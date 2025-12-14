---
title: "Huff 详细使用示例"
date: 2025-05-16
draft: false
tags: ["Web3", "lanuage", "Huff"]
showToc: true
---


# 简介

**Huff by Example** 旨在详细解释 Huff 语言的各项特性，并通过代码示例说明每个特性的使用方法、使用时机、使用场景以及设计目的。文中的代码片段附有丰富注释，但本节假设读者对 **EVM** 有一定基础

---

## 定义接口 (Defining your Interface)

在 Huff 中定义接口不是必需的步骤，但可以为以下两个目的使用：

1. **作为 `__FUNC_SIG` 和 `__EVENT_HASH` 内置函数的参数**
2. **生成 Solidity 接口 / 合约 ABI**

**说明**：

* **函数类型**：可以是 `view`、`pure`、`payable` 或 `nonpayable`
* **函数接口**：仅建议为外部可调用函数定义
* **事件（Events）**：可以包含索引值（indexed）和非索引值（non-indexed）

**示例**:

```huff
#define function testFunction(uint256, bytes32) view returns (bytes memory)

#define event TestEvent(address indexed, uint256)
```

* `testFunction`：定义了一个接受 `uint256` 和 `bytes32` 参数的只读函数，返回 `bytes` 类型
* `TestEvent`：定义了一个事件，包含一个索引地址和一个 `uint256` 参数

---


## 常量 (Constants)

在 Huff 合约中，**常量不会存储在合约的 storage 中**，而是在编译时即可在合约内调用

常量可以是以下两种类型：

1. **字节（bytes）**：最大 32 字节
2. **FREE_STORAGE_POINTER**：表示合约中未使用的存储槽（storage slot）


**使用方法**:

* 将常量压入堆栈时，使用 **方括号** 表示法：`[CONSTANT]`



**示例**:

* 常量声明

```huff
#define constant NUM = 0x420
#define constant HELLO_WORLD = 0x48656c6c6f2c20576f726c6421
#define constant FREE_STORAGE = FREE_STORAGE_POINTER()
```

* 常量使用

假设常量 `NUM` 的值为 `0x420`：

```text
// [] - 空堆栈
[NUM] // [0x420] - 常量的值被压入堆栈
```

* 方括号 `[NUM]` 表示将常量值推入堆栈
* 使用常量不会占用 storage 空间(存在字节码中)，节省合约存储成本

---

## 自定义错误 (Custom Errors)

在 Huff 中，可以定义 **自定义错误**，并通过 `__ERROR` 内置函数将左填充的 4 字节错误选择器压入堆栈



**示例**:

* 定义自定义错误

```huff
#define error PanicError(uint256)
#define error Error(string)
```
* 使用自定义错误

```huff
#define macro PANIC() = takes (1) returns (0) {
    // 输入堆栈: [panic_code]
    __ERROR(PanicError)   // 堆栈变为 [panic_error_selector, panic_code]
    0x00 mstore           // 堆栈变为 [panic_code]
    0x04 mstore           // 堆栈变为空 []
    0x24 0x00 revert
}

#define macro REQUIRE() = takes (3) returns (0) {
    // 输入堆栈: [condition, message_length, message]
    continue jumpi        // 条件成立则跳转 continue [message_length, message]

    __ERROR(Error)        // 条件不成立: 堆栈变为 [error_selector, message_length, message]
    0x00 mstore           // [message_length, message]
    0x20 0x04 mstore      // [message_length, message]
    0x24 mstore           // [message]
    0x44 mstore           // []

    0x64 0x00 revert      // 抛出错误

    continue:
        pop               // 条件成立则清空堆栈 []
}
```
---

好的，我帮你整理成中文笔记风格文档：

---

## 跳转标签 (Jump Labels)

跳转标签是 Huff 提供的一种简单抽象，用于简化对 **JUMPDEST** 的定义和引用，让开发者更方便地控制跳转位置


**示例**:

```huff
#define macro MAIN() = takes (0) returns (0) {
    // 将 "Hello, World!" 存入内存
    0x48656c6c6f2c20576f726c6421
    0x00 mstore       // 堆栈: ["Hello, World!"]

    // 跳转到 success 标签，跳过 revert 语句
    success           // 堆栈: [success_label_pc, "Hello, World!"]
    jump              // 执行跳转后堆栈: ["Hello, World!"]

    // 如果执行到此处，则回滚
    0x00 0x00 revert

    // 标签定义
    // 标签在宏或函数中定义，由一个单词加冒号表示
    // 注意：虽然看起来像缩进块，但标签只是字节码中的跳转目标
    // 标签下方的操作会被执行，除非程序计数器被修改，或者执行被以下 opcode 中止：
    // `revert`、`return`、`stop` 或 `selfdestruct`
    success:
        0x00 mstore
        0x20 0x00 return
}
```

**说明**:

* `success:` 是跳转目标标签
* `jump` 将程序计数器移动到指定标签位置
* 标签只是字节码跳转的目的地，不会自动形成作用域
* 标签下方的指令会被执行，除非被 `revert`、`return` 等操作中止

---

好的，我帮你整理成中文笔记风格文档，条理清晰、适合学习参考：

---

## 宏（Macros）与函数（Functions）

Huff 提供了两种方式来组织字节码：**宏（Macros）**和**函数（Functions）**。理解它们的区别以及使用场景非常重要。

两者的定义方式类似，都可以接收可选参数，并跟随 `takes` 和 `returns` 关键字，分别表示宏/函数从堆栈读取的输入数量和输出数量。如果不使用 `takes` 或 `returns`，默认值为 0。

```huff
#define <macro|fn> TEST(err) = takes (1) returns (3) {
    // ...
}
```

### 宏（Macros）

* Huff 开发者大多会使用宏
* 每次调用宏时，宏内的代码会被**直接嵌入调用处**
* 优点：运行时 gas 消耗低，不需要跳转
* 缺点：如果宏被大量使用，会快速增加合约字节码体积



**构造函数和 MAIN 宏**

* **MAIN** 和 **CONSTRUCTOR** 是两个特殊宏
* **MAIN**：合约调用时的 fallback，通常是控制流的入口
* **CONSTRUCTOR**（可选）：合约部署时初始化使用，输入在编译时提供

> 默认情况下，CONSTRUCTOR 会添加启动代码，将编译后的 MAIN 宏作为合约的运行时代码返回。如果 CONSTRUCTOR 使用了 `RETURN` opcode，则不会添加启动代码，合约将使用 CONSTRUCTOR 返回的代码进行部署


**宏参数**:

* 宏可以接受参数，用于宏内部调用或作为引用传入
* 参数类型可以是：**标签（label）、操作码（opcode）、字面量（literal）、常量（constant）**
* 宏在编译时被内联，参数在运行时不会被重新计算，也会被内联



**示例**：

```huff
// 定义合约接口
#define function addWord(uint256) pure returns (uint256)

// 获取一个空闲 storage 槽来存储 owner
#define constant OWNER = FREE_STORAGE_POINTER()

// 定义事件
#define event WordAdded(uint256 initial, uint256 increment)

// 宏：发出添加 word 的事件
#define macro emitWordAdded(increment) = takes (1) returns (0) {
    // 输入堆栈: [initial]
    <increment>              // [increment, initial]
    __EVENT_HASH(WordAdded)  // [sig, increment, initial]
    0x00 0x00                // [mem_start, mem_end, sig, increment, initial]
    log3                     // []
}

// 宏：仅限 owner 执行
#define macro ONLY_OWNER() = takes (0) returns (0) {
    caller                   // [msg.sender]
    [OWNER] sload            // [owner, msg.sender]
    eq                       // [owner == msg.sender]
    is_owner jumpi           // []

    // 如果不是 owner，则 revert
    0x00 0x00 revert

    is_owner:
}

// 宏：向 uint 中添加 word（32 bytes）
#define macro ADD_WORD() = takes (1) returns (1) {
    // 输入堆栈: [input_num]

    // 检查调用者是否为 owner
    ONLY_OWNER()

    // 调用 emitWordAdded 宏
    dup1                     // [input_num, input_num]
    emitWordAdded(0x20)      // [input_num]

    // 将 0x20 添加到输入数字
    0x20                     // [0x20, input_num]
    add                      // [0x20 + input_num]

    // 输出堆栈: [0x20 + input_num]
}

// MAIN 宏
#define macro MAIN() = takes (0) returns (0) {
    // 从 calldata 获取函数签名
    0x00 calldataload        // [calldata @ 0x00]
    0xE0 shr                 // [func_sig]

    // 检查是否匹配 addWord 函数
    __FUNC_SIG(addWord)      // [func_sig(addWord), func_sig]
    eq                       // [func_sig(addWord) == func_sig]
    add_word jumpi           // []

    // 未匹配则 revert
    0x00 0x00 revert

    // 跳转标签 add_word
    add_word:
        0x04 calldataload    // [input_num]
        ADD_WORD()           // [result]
        0x00 mstore          // []
        0x20 0x00 return
}
```

**说明**:

* 使用宏可以高效组织代码并内联，减少运行时 gas
* `MAIN` 宏处理函数选择器和调用逻辑
* `ONLY_OWNER` 和 `emitWordAdded` 宏示范了宏嵌套与事件发射
* 通过跳转标签 `add_word` 实现函数分支



### 函数（Functions）

Huff 的函数看起来和宏非常相似，但行为有所不同：

* **宏（Macros）**：编译时内联，调用处直接复制代码
* **函数（Functions）**：编译器会把函数代码移动到**运行时代码末尾**，调用处会插入跳转到函数代码，以及返回点的跳转标签

> 优点：可以减少合约字节码大小，尤其适合重复逻辑较多的大合约
> 缺点：每次调用函数会增加少量运行时 gas（大约 22 + n_inputs * 3 + n_outputs * 3 gas）


**函数参数**:

* 函数可以接受参数，用于函数内部调用或作为引用
* 参数类型可以是：**标签（label）、操作码（opcode）、字面量（literal）、常量（constant）**
* 因为函数代码会放到字节码末尾，参数在运行时不重新计算，而是**编译时内联**


**示例**:

```huff
#define macro MUL_DIV_DOWN_WRAPPER() = takes (0) returns (0) {
    0x44 calldataload // [denominator]  从 calldata 加载分母
    0x24 calldataload // [y, denominator]  从 calldata 加载 y
    0x04 calldataload // [x, y, denominator]  从 calldata 加载 x
    
    // 与宏不同，函数的代码不会被直接内联到调用处，
    // 它会被放置到合约运行时代码的末尾，
    // 并在这里插入跳转到函数代码的指令，以及跳转回调用点的 JUMPDEST
    //
    // 编译器会查看函数需要的栈输入数量 N，
    // 并生成一个长度为 N 的 SWAP 指令数组，按降序排列：
    // SWAP1 + N - 1 -> SWAP1
    //
    // 对于本次函数调用，我们有三个输入，因此需要三条 SWAP 指令：
    // swap3、swap2、swap1。返回的跳转目标必须在栈输入的下方，
    // 同时输入仍然保持正确顺序
    //
    // 初始栈状态：
    // [return_pc, x, y, denominator]
    // 执行 swap3 后：
    // [denominator, x, y, return_pc]
    // 执行 swap2 后：
    // [y, x, denominator, return_pc]
    // 执行 swap1 后：
    // [x, y, denominator, return_pc]
    //
    // 完成后，编译器会插入：
    // 1. 跳转到函数起始 JUMPDEST 的 JUMP 指令
    // 2. 返回调用点的 JUMPDEST
    //
    // 调用函数时实际插入的字节码：
    // PUSH2 return_pc   -- 压入返回地址
    // <num_inputs swap ops> -- 对输入参数进行 SWAP
    // PUSH2 func_start_pc -- 压入函数起始位置
    // JUMP               -- 跳转到函数
    // JUMPDEST           -- 返回调用点的跳转目标
    MUL_DIV_DOWN(err) // 调用函数，栈上剩余 [result]

    // 返回结果
    0x00 mstore
    0x20 0x00 return

    err:
        0x00 0x00 revert
}

#define fn MUL_DIV_DOWN(err) = takes (3) returns (1) {
    // 编译器在此插入一个 JUMPDEST 指令
    // 初始栈状态：[x, y, denominator, return_pc]

    // 函数主体代码 ...

    // 因为编译器知道函数返回 N 个栈元素，
    // 它会在函数末尾按升序插入 N 条 SWAP 指令：
    // SWAP1 -> SWAP1 + N - 1
    // 目的是把 return_pc 放到栈顶，以供 JUMP 使用
    //
    // 假设返回一个元素 result：
    // 初始栈状态：[result, return_pc]
    // 执行 swap1 后：[return_pc, result]
    //
    // 最终函数字节码结构：
    // 👇 func_start_pc
    // JUMPDEST           [x, y, denominator, return_pc]
    // 函数代码执行完    [result, return_pc]
    // SWAP1              [return_pc, result]
    // JUMP               跳回调用点，栈上为 [result]
}

```

**调用过程解析**:

1. **包装宏 `MUL_DIV_DOWN_WRAPPER`**

   * 从 calldata 中读取参数（x、y、denominator）
   * 调用 `MUL_DIV_DOWN` 函数
   * 函数调用不是直接复制代码，而是在末尾生成函数字节码，并在调用处插入：

     * 返回点标签（return_pc）
     * 若干 SWAP 操作，使参数顺序正确
     * 跳转到函数起始点（JUMP）

2. **函数 `MUL_DIV_DOWN`**

   * 起始处自动插入 JUMPDEST
   * 执行函数逻辑
   * 返回前编译器自动交换堆栈，将 return_pc 移到栈顶
   * 执行 JUMP 返回调用点

---

### 核心区别总结

| 特性    | 宏（Macro） | 函数（Function）         |
| ----- | -------- | -------------------- |
| 代码位置  | 内联到调用处   | 放到字节码末尾              |
| 调用开销  | 无额外跳转    | 需跳转 + return_pc SWAP |
| 字节码大小 | 容易增加     | 复用逻辑可减少字节码           |
| 适用场景  | 小逻辑、高频调用 | 重复逻辑、较大合约            |

---


## 内置函数（Builtin Functions）

Huff 编译器提供了若干内置函数，用于简化常见操作

### `#__FUNC_SIG(<func_def|string>)`

在编译时，`__FUNC_SIG` 会被替换为：

```huff
PUSH4 function_selector
```

其中 `function_selector` 是传入的函数定义或字符串对应的 4 字节函数选择器。如果传入的是字符串，它必须是有效的函数签名，例如：

```text
"test(address, uint256)"
```


### `#__EVENT_HASH(<event_def|string>)`

在编译时，`__EVENT_HASH` 会被替换为：

```huff
PUSH32 event_hash
```

其中 `event_hash` 是传入的事件定义或字符串的哈希选择器。如果传入的是字符串，它必须是有效事件签名，例如：

```text
"TestEvent(uint256, address indexed)"
```

### `#__ERROR(<error_def>)`

在编译时，`__ERROR` 会被替换为：

```huff
PUSH32 error_selector
```

其中 `error_selector` 是传入错误定义的左填充 4 字节选择器

---

### `#__RIGHTPAD(<literal>)`

在编译时，`__RIGHTPAD` 会被替换为：

```huff
PUSH32 padded_literal
```

其中 `padded_literal` 是传入字面量右填充 32 字节后的值

---

### `#__codesize(MACRO|FUNCTION)`

将传入宏或函数的字节码长度压入栈顶

---

### `#__tablestart(TABLE)` 和 `#__tablesize(TABLE)`

这两个函数与跳转表（Jump Tables）相关，具体内容将在下一节说明


**示例**:

```huff
// 定义函数
#define function test1(address, uint256) nonpayable returns (bool)
#define function test2(address, uint256) nonpayable returns (bool)

// 定义事件
#define event TestEvent1(address, uint256)
#define event TestEvent2(address, uint256)

// 定义宏 TEST1，触发 TestEvent1
#define macro TEST1() = takes (0) returns (0) {
    0x00 0x00                // [address, uint]
    __EVENT_HASH(TestEvent1) // [sig, address, uint]
    0x00 0x00                // [mem_start, mem_end, sig, address, uint]
    log3                     // []
}

// 定义宏 TEST2，触发 TestEvent2
#define macro TEST2() = takes (0) returns (0) {
    0x00 0x00                // [address, uint]
    __EVENT_HASH(TestEvent2) // [sig, address, uint]
    0x00 0x00                // [mem_start, mem_end, sig, address, uint]
    log3                     // []
}

// 主宏 MAIN，根据函数选择器调用相应函数
#define macro MAIN() = takes (0) returns (0) {
    // 从 calldata 获取函数选择器
    0x00 calldataload 0xE0 shr

    // 比较函数选择器并跳转
    dup1 __FUNC_SIG(test1) eq test1 jumpi
    dup1 __FUNC_SIG(test2) eq test2 jumpi

    // 若无匹配函数则回滚
    0x00 0x00 revert

    test1:
        TEST1()

    test2:
        TEST2()
}
```
---

下面是你提供的 Huff **Jump Tables** 内容的完整中文翻译与说明：

---

## 跳转表（Jump Tables）

跳转表是在 Huff 合约中实现类似 `switch/case` 的便捷方法。每个跳转表包含若干 `jumpdest` 程序计数器（PC），这些 PC 被写入到合约的字节码中。你可以将这些 `jumpdest` 复制到内存中，然后通过查找特定内存指针处的 `jumpdest` 来选择对应的分支（例如：`0x00 = case 1`，`0x20 = case 2` 等）。这样可以通过一次跳转实现多分支，而无需多次条件跳转。

**跳转表类型**:

Huff 中有两种跳转表：

1. **常规跳转表（Regular Jump Tables）**

   * 每个 `jumpdest` 用完整的 32 字节存储
   * 从内存中获取 PC 较快，但复制到内存成本较高

2. **打包跳转表（Packed Jump Tables）**

   * 每个 `jumpdest` 仅占 2 字节
   * 复制到内存成本低，但取出 PC 时需要额外的位移操作，成本较高

**内置函数**:

* `#__tablestart(TABLE)`
  将传入表的起始程序计数器（PC）压入栈顶

* `#__tablesize(TABLE)`
  将传入表的字节码长度压入栈顶


**示例**:

```huff
// 定义函数
#define function switchTest(uint256) pure returns (uint256)

// 定义一个包含 4 个 jumpdest 的跳转表
#define jumptable SWITCH_TABLE {
    jump_one jump_two jump_three jump_four
}

#define macro SWITCH_TEST() = takes (0) returns (0) {
    // 将跳转表复制到内存 @0x00
    __tablesize(SWITCH_TABLE)   // [table_size]
    __tablestart(SWITCH_TABLE)  // [table_start, table_size]
    0x00
    codecopy

    0x04 calldataload           // [input_num]

    // 如果 input_num 不在 [0,3] 范围内则回滚
    dup1                        // [input_num, input_num]
    0x03 lt                     // [3 < input_num, input_num]
    err jumpi                   

    // 常规跳转表存储 jumpdest PC 为完整字
    // 通过将输入数字乘以 32 来确定跳转标签
    0x20 mul                    // [0x20 * input_num]
    mload                       // [pc]
    jump                        // []

    jump_one:
        0x100 0x00 mstore
        0x20 0x00 return
    jump_two:
        0x200 0x00 mstore
        0x20 0x00 return
    jump_three:
        0x300 0x00 mstore
        0x20 0x00 return
    jump_four:
        0x400 0x00 mstore
        0x20 0x00 return
    err:
        0x00 0x00 revert
}

#define macro MAIN() = takes (0) returns (0) {
    // 获取函数选择器
    0x00 calldataload 0xE0 shr
    dup1 __FUNC_SIG(switchTest) eq switch_test jumpi

    // 无匹配函数则回滚
    0x00 0x00 revert

    switch_test:
        SWITCH_TEST()
}
```

---

下面是你提供内容的中文翻译笔记版：

---

## 代码表（Code Tables）

代码表用于存放原始字节码（raw bytecode）。编译器会把表中的代码放在运行时代码的末尾，前提是它们在合约中被引用过

**示例**:

```huff
#define table CODE_TABLE {
    0x604260005260206000F3
}
```

---

## Huff 测试（Huff Tests）

Huff 编译器内置了一个简化的测试框架，用于：

* 创建断言（assertions）
* 对宏（macros）和函数（functions）进行 Gas 使用分析

注意：huff-rs 的测试框架功能非常有限，不足以替代 **foundry-huff** 等成熟工具

* 如果你的合约是生产级的，建议同时使用 **Foundry** 和 **Huff Tests**
* 如果你只是用 Huff 学习 EVM 或调试逻辑，Huff Tests 可以提供轻量级体验

**运行方式**:

测试可以通过 CLI 的 `test` 子命令运行。更多信息请参考 [CLI 文档](/web3/codelanuage/huff/cli)



### 装饰器（Decorators）

测试中可以使用 **装饰器** 来修改交易环境

* 装饰器位于测试函数上方，格式如下：

```huff
#[flag_a(inputs...), flag_b(inputs...)]
```

可用装饰器示例：

* `calldata`：设置交易的 calldata，接受单个字节串
* `value`：设置交易的 callvalue，接受单个字面量（literal）。

**示例**:

```huff
#include "huffmate/utils/Errors.huff"

#define macro ADD_TWO() = takes (2) returns (1) {
    // 输入栈:  [a, b]
    add           // [a + b]
    // 返回栈: [a + b]
}

#[calldata("0x0000000000000000000000000000000000000000000000000000000000000001"), value(0x01)]
#define test MY_TEST() = {
    0x00 calldataload   // [0x01]
    callvalue           // [0x01, 0x01]
    eq ASSERT()
}
```
---







