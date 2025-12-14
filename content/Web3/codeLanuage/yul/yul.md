---
title: "Yul 使用指南"
date: 2025-02-16
draft: false
tags: ["Web3", "lanuage", "Yul"]
showToc: true
---

# 简介

**Yul**（早期称为 JULIA 或 IULIA）是一种中间语言，可编译到多种后端，包括：

* Ethereum Virtual Machine (EVM) 1.0
* EVM 1.5
* 计划支持的 eWASM

它旨在作为这些平台的通用标准。Yul 已经可以在 Solidity 中作为 **内联汇编** 使用，未来 Solidity 编译器甚至可能将 Yul 用作默认的中间语言。Yul 的设计也使得为其构建高级优化器变得容易

**核心特性**:

1. **核心组件**

   * 函数（function）
   * 代码块（block）
   * 变量（variable）
   * 字面量（literal）
   * 循环（for）
   * 条件语句（if / switch）
   * 表达式与变量赋值

2. **强类型系统**

   * 变量和字面量必须带类型前缀
   * 支持类型：`bool`, `u8`, `s8`, `u32`, `s32`, `u64`, `s64`, `u128`, `s128`, `u256`, `s256`

3. **操作符与内置函数**

   * Yul 本身不提供操作符
   * 对于 EVM，操作码（opcode）作为内置函数提供
   * 如果后端平台不同，可以重新实现这些函数


**示例**：

- 递归实现

```yul
{
    function power(base:u256, exponent:u256) -> result:u256
    {
        switch exponent
        case 0:u256 { result := 1:u256 }
        case 1:u256 { result := base }
        default:
        {
            result := power(mul(base, base), div(exponent, 2:u256))
            switch mod(exponent, 2:u256)
                case 1:u256 { result := mul(base, result) }
        }
    }
}
```

- 循环实现

```yul
{
    function power(base:u256, exponent:u256) -> result:u256
    {
        result := 1:u256
        for { let i := 0:u256 } lt(i, exponent) { i := add(i, 1:u256) }
        {
            result := mul(result, base)
        }
    }
}
```

> 注：循环实现依赖 EVM 的 `lt` 和 `add` 操作码。

---

## 语法概览
>  Yul 代码通常放置在一个 [Yul 对象](#对象说明)* 中

### 代码块与语句

```text
代码块 = '{' 语句* '}'
语句 =
    代码块 |
    函数定义 |
    变量声明 |
    赋值 |
    表达式 |
    Switch |
    For 循环 |
    循环中断
```
**示例**:
```yul
{
    // 一个包含多个语句的代码块
    let x:u256 := 10:u256
    x := add(x, 5:u256)       // 赋值（单个标识符 := 表达式）
    {
        // 嵌套块
        let y:u256 := mul(x, 2:u256)
    }
}
```

### 函数定义

```text
函数定义 =
    'function' 标识符 '(' 带类型的标识符列表? ')'
    ( '->' 带类型的标识符列表 )? 代码块
```

**示例**:
```yul
{
    function addTwo(a:u256, b:u256) -> r:u256 {
        r := add(a, b)
    }

    // 调用
    let sum:u256 := addTwo(3:u256, 4:u256)
}

```

### 变量声明与赋值

```text
变量声明 =
    'let' 带类型的标识符列表 ( ':=' 表达式 )?

赋值 =
    标识符列表 ':=' 表达式
```

**示例**:
```yul
{
    // 声明带类型标识符（不赋初值）
    let counter:u256
    // 声明时赋初值
    let total:u256 := 0:u256

    total := add(total, 1:u256)
}
```

### 表达式

```text
表达式 =
    函数调用 | 标识符 | 字面量
函数调用 =
    标识符 '(' ( 表达式 ( ',' 表达式 )* )? ')'
```

**示例**:
```yul
{
    // 标识符作为表达式：x
    let x:u256 := 5:u256

    // 字面量作为表达式：直接使用 10:u256
    let y:u256 := 10:u256

    // 函数调用作为表达式
    function timesTwo(a:u256) -> r:u256 { r := mul(a, 2:u256) }
    let z:u256 := timesTwo(y)
}
```

### 条件语句

```text
If 条件语句 = 'if' 表达式 代码块
Switch 条件语句 =
    'switch' 表达式 Case* ( 'default' 代码块 )?
Case = 'case' 字面量 代码块
```

**示例**:
```yul
{
    let a:u256 := 3:u256
    if lt(a, 5:u256) {
        // 当 lt(a,5) 为真时执行
        a := add(a, 1:u256)
    }
}

{
    let flag:bool := true:bool
    switch flag
    case true:bool {
        // flag 为 true 时
        let res:u256 := 1:u256
    }
    case false:bool {
        // flag 为 false 时
        let res:u256 := 0:u256
    }
    // no default，因为 true/false 已覆盖所有可能值
}
```


### 循环语句

```text
For 循环 = 'for' 代码块 条件表达式 代码块 代码块
循环中断 = 'break' | 'continue'
```

**示例**:
```yul
{
    let result:u256 := 1:u256
    for { let i := 0:u256 } lt(i, 4:u256) { i := add(i, 1:u256) } {
        // 循环体
        result := mul(result, 2:u256)
    }
    // 执行后 result == 16 (2^4)
}

{
    for { let i := 0:u256 } lt(i, 10:u256) { i := add(i, 1:u256) } {
        if eq(i, 3:u256) { continue }   // 跳过 i==3 那次循环
        if eq(i, 6:u256) { break }      // 遇到 i==6 跳出循环
        // 其余情况执行
    }
}

```

### 标识符与类型

```text
标识符 = [a-zA-Z_$][a-zA-Z_0-9]*
类型名 = 标识符 | 内置类型名
内置类型名 = 'bool' | [us](8|32|64|128|256)
带类型的标识符列表 = 标识符 ':' 类型名 ( ',' 标识符 ':' 类型名 )*
```
### 字面量

```text
字面量 = (数字字面量 | 字符串字面量 | 十六进制字面量 | true | false) ':' 类型名
数字字面量 = 十六进制数字 | 十进制数字
十六进制字面量 = 'hex' '"'([0-9a-fA-F]{2})*'"' | '\''([0-9a-fA-F]{2})*'\''
字符串字面量 = '"' ([^"\r\n\\] | '\\' .)* '"'
十六进制数字 = '0x'[0-9a-fA-F]+
十进制数字 = [0-9]+
```

### 语法限制与规则

* `switch` 必须至少有一个 `case`
* 表达式求值规则：

  * 标识符和字面量 → 一个值
  * 函数调用 → 返回值数量
  * 变量声明和赋值 → 右侧表达式数量必须与左侧变量数量一致
* `continue` 与 `break`：

  * 只能用于循环体
  * 必须与循环在同一函数或顶层
* `for` 循环条件部分 → 必须求值为一个值
* 字面量不可超出类型最大值（最大 256 位）

### 作用域规则

* 变量作用域与 **块** 和声明紧密绑定
* 标识符可见性：

  * 在定义的块及其子块中可见
  * `for` 循环 init 部分定义的标识符 → 在循环其他部分可见
* 函数参数和返回值 → 在函数体内可见
* 不允许 **shadowing**（同名变量覆盖）
* 函数可在声明前引用，变量必须先声明后使用
* 函数内无法访问外部变量


### 类型转换

* **不支持隐式类型转换** → 需显式调用函数
* 支持截取式转换：

  * 类型：`bool`, `u32`, `u64`, `u256`, `s256`
  * 原型：`<input_type>to<output_type>(x:<input_type>) -> y:<output_type>`
* 示例：

```yul
u32tobool(x:u32) -> y:bool
booltou32(x:bool) -> y:u32
u256tou32(x:u256) -> y:u32
s256tou256(x:s256) -> y:u256
```
---
好的，我帮你把这些 **Yul 低级函数**整理成一个清晰的笔记表格，按功能分类，方便查阅和记忆：

---

### 低级函数

| 分类        | 函数                                                                                     | 功能说明                  | 示例                                                                                           |
| --------- | -------------------------------------------------------------------------------------- | --------------------- | -------------------------------------------------------------------------------------------- |
| **逻辑操作**  | `not(x:bool) -> z:bool`                                                                | 逻辑非                   | `let z:bool := not(true:bool)`                                                               |
|           | `and(x:bool, y:bool) -> z:bool`                                                        | 逻辑与                   | `let z:bool := and(true:bool, false:bool)`                                                   |
|           | `or(x:bool, y:bool) -> z:bool`                                                         | 逻辑或                   | `let z:bool := or(true:bool, false:bool)`                                                    |
|           | `xor(x:bool, y:bool) -> z:bool`                                                        | 异或                    | `let z:bool := xor(true:bool, false:bool)`                                                   |
| **算术操作**  | `addu256(x:u256, y:u256) -> z:u256`                                                    | x + y                 | `let z:u256 := addu256(5:u256, 3:u256)`                                                      |
|           | `subu256(x:u256, y:u256) -> z:u256`                                                    | x - y                 | `let z:u256 := subu256(5:u256, 3:u256)`                                                      |
|           | `mulu256(x:u256, y:u256) -> z:u256`                                                    | x * y                 | `let z:u256 := mulu256(5:u256, 3:u256)`                                                      |
|           | `divu256(x:u256, y:u256) -> z:u256`                                                    | x / y                 | `let z:u256 := divu256(10:u256, 2:u256)`                                                     |
|           | `divs256(x:s256, y:s256) -> z:s256`                                                    | 有符号除法                 | `let z:s256 := divs256(-10:s256, 2:s256)`                                                    |
|           | `modu256(x:u256, y:u256) -> z:u256`                                                    | x % y                 | `let z:u256 := modu256(10:u256, 3:u256)`                                                     |
|           | `mods256(x:s256, y:s256) -> z:s256`                                                    | 有符号取模                 | `let z:s256 := mods256(-10:s256, 3:s256)`                                                    |
|           | `signextendu256(i:u256, x:u256) -> z:u256`                                             | 从第 (i*8+7) 位开始符号扩展    | `let z:u256 := signextendu256(1:u256, 0xff:u256)`                                            |
|           | `expu256(x:u256, y:u256) -> z:u256`                                                    | x 的 y 次方              | `let z:u256 := expu256(2:u256, 8:u256)`                                                      |
|           | `addmodu256(x:u256, y:u256, m:u256) -> z:u256`                                         | (x + y) % m           | `let z:u256 := addmodu256(5:u256, 7:u256, 6:u256)`                                           |
|           | `mulmodu256(x:u256, y:u256, m:u256) -> z:u256`                                         | (x * y) % m           | `let z:u256 := mulmodu256(5:u256, 7:u256, 6:u256)`                                           |
|           | `ltu256(x:u256, y:u256) -> z:bool`                                                     | x < y                 | `let z:bool := ltu256(2:u256, 3:u256)`                                                       |
|           | `gtu256(x:u256, y:u256) -> z:bool`                                                     | x > y                 | `let z:bool := gtu256(5:u256, 3:u256)`                                                       |
|           | `sltu256(x:s256, y:s256) -> z:bool`                                                    | 有符号 x < y             | `let z:bool := sltu256(-2:s256, 3:s256)`                                                     |
|           | `sgtu256(x:s256, y:s256) -> z:bool`                                                    | 有符号 x > y             | `let z:bool := sgtu256(2:s256, -3:s256)`                                                     |
|           | `equ256(x:u256, y:u256) -> z:bool`                                                     | x == y                | `let z:bool := equ256(5:u256, 5:u256)`                                                       |
|           | `iszerou256(x:u256) -> z:bool`                                                         | x == 0                | `let z:bool := iszerou256(0:u256)`                                                           |
|           | `notu256(x:u256) -> z:u256`                                                            | 按位非 (~x)              | `let z:u256 := notu256(0xff:u256)`                                                           |
|           | `andu256(x:u256, y:u256) -> z:u256`                                                    | 按位与                   | `let z:u256 := andu256(0xf0:u256, 0x0f:u256)`                                                |
|           | `oru256(x:u256, y:u256) -> z:u256`                                                     | 按位或                   | `let z:u256 := oru256(0xf0:u256, 0x0f:u256)`                                                 |
|           | `xoru256(x:u256, y:u256) -> z:u256`                                                    | 按位异或                  | `let z:u256 := xoru256(0xf0:u256, 0x0f:u256)`                                                |
|           | `shlu256(x:u256, y:u256) -> z:u256`                                                    | 逻辑左移                  | `let z:u256 := shlu256(1:u256, 4:u256)`                                                      |
|           | `shru256(x:u256, y:u256) -> z:u256`                                                    | 逻辑右移                  | `let z:u256 := shru256(16:u256, 4:u256)`                                                     |
|           | `saru256(x:u256, y:u256) -> z:u256`                                                    | 算术右移                  | `let z:u256 := saru256(16:u256, 4:u256)`                                                     |
|           | `byte(n:u256, x:u256) -> v:u256`                                                       | 第 n 字节                | `let v:u256 := byte(1:u256, 0x1234:u256)`                                                    |
| **内存与存储** | `mload(p:u256) -> v:u256`                                                              | 读 32 字节内存             | `let x:u256 := mload(0x40:u256)`                                                             |
|           | `mstore(p:u256, v:u256)`                                                               | 写 32 字节内存             | `mstore(0x40:u256, 0x1234:u256)`                                                             |
|           | `mstore8(p:u256, v:u256)`                                                              | 写单字节                  | `mstore8(0x40:u256, 0xff:u256)`                                                              |
|           | `sload(p:u256) -> v:u256`                                                              | 读取存储                  | `let x:u256 := sload(0x01:u256)`                                                             |
|           | `sstore(p:u256, v:u256)`                                                               | 写存储                   | `sstore(0x01:u256, 0x10:u256)`                                                               |
|           | `msize() -> size:u256`                                                                 | 内存已使用大小               | `let size:u256 := msize()`                                                                   |
| **执行控制**  | `create(v:u256, p:u256, s:u256)`                                                       | 创建新合约                 | `let addr := create(0:u256, 0x40:u256, 0x20:u256)`                                           |
|           | `call(g:u256, a:u256, v:u256, in:u256, insize:u256, out:u256, outsize:u256) -> r:u256` | 调用外部合约                | `let success := call(100000:u256, addr, 0:u256, 0x40:u256, 0x20:u256, 0x60:u256, 0x20:u256)` |
|           | `delegatecall(...) -> r:u256`                                                          | 保留 caller / callvalue | 示例同 `call`                                                                                   |
|           | `abort()`                                                                              | 终止执行                  | `abort()`                                                                                    |
|           | `return(p:u256, s:u256)`                                                               | 返回内存数据                | `return(0x40:u256, 0x20:u256)`                                                               |
|           | `revert(p:u256, s:u256)`                                                               | 回退执行                  | `revert(0x40:u256, 0x20:u256)`                                                               |
|           | `selfdestruct(a:u256)`                                                                 | 销毁合约                  | `selfdestruct(addr)`                                                                         |
|           | `log0(p:u256, s:u256)` ~ `log4(...)`                                                   | 产生日志                  | `log1(0x40:u256, 0x20:u256, topic)`                                                          |
| **状态查询**  | `blockcoinbase() -> address:u256`                                                      | 当前矿工                  | `let m := blockcoinbase()`                                                                   |
|           | `blockdifficulty() -> difficulty:u256`                                                 | 区块难度                  | `let d := blockdifficulty()`                                                                 |
|           | `blockgaslimit() -> limit:u256`                                                        | 区块 gas 限制             | `let l := blockgaslimit()`                                                                   |
|           | `blockhash(b:u256) -> hash:u256`                                                       | 最近 256 个区块哈希          | `let h := blockhash(10:u256)`                                                                |
|           | `blocknumber() -> block:u256`                                                          | 当前区块号                 | `let n := blocknumber()`                                                                     |
|           | `blocktimestamp() -> timestamp:u256`                                                   | 区块时间戳                 | `let t := blocktimestamp()`                                                                  |
|           | `txorigin() -> address:u256`                                                           | 交易发送方                 | `let o := txorigin()`                                                                        |
|           | `txgasprice() -> price:u256`                                                           | gas 价格                | `let p := txgasprice()`                                                                      |
|           | `gasleft() -> gas:u256`                                                                | 剩余 gas                | `let g := gasleft()`                                                                         |
|           | `balance(a:u256) -> v:u256`                                                            | 地址余额                  | `let b := balance(addr)`                                                                     |
|           | `this() -> address:u256`                                                               | 当前合约地址                | `let me := this()`                                                                           |
|           | `caller() -> address:u256`                                                             | 调用者                   | `let c := caller()`                                                                          |
|           | `callvalue() -> v:u256`                                                                | 当前调用发送的 wei           | `let v := callvalue()`                                                                       |
|           | `calldataload(p:u256) -> v:u256`                                                       | calldata 32 字节        | `let v := calldataload(0x00:u256)`                                                           |
|           | `calldatasize() -> v:u256`                                                             | calldata 大小           | `let s := calldatasize()`                                                                    |
|           | `calldatacopy(t:u256, f:u256, s:u256)`                                                 | 拷贝 calldata 到内存       | `calldatacopy(0x40:u256, 0x00:u256, 0x20:u256)`                                              |
|           | `codesize() -> size:u256`                                                              | 当前代码大小                | `let sz := codesize()`                                                                       |
|           | `codecopy(t:u256, f:u256, s:u256)`                                                     | 拷贝代码到内存               | `codecopy(0x40:u256, 0x00:u256, 0x20:u256)`                                                  |
|           | `extcodesize(a:u256) -> size:u256`                                                     | 外部地址代码大小              | `let sz := extcodesize(addr)`                                                                |
|           | `extcodecopy(a:u256, t:u256, f:u256, s:u256)`                                          | 外部地址拷贝代码              | `extcodecopy(addr, 0x40:u256, 0x00:u256, 0x20:u256)`                                         |
| **其他操作**  | `discard(unused:bool)`                                                                 | 丢弃值                   | `discard(true:bool)`                                                                         |
|           | `discardu256(unused:u256)`                                                             | 丢弃值                   | `discardu256(0x123:u256)`                                                                    |
|           | `splitu256tou64(x:u256) -> (x1:u64,x2:u64,x3:u64,x4:u64)`                              | 拆分 u256 为四个 u64       | `let x1,x2,x3,x4 := splitu256tou64(0x123456:u256)`                                           |
|           | `combineu64tou256(x1:u64,x2:u64,x3:u64,x4:u64) -> x:u256`                              | 四个 u64 合并成 u256       | `let x := combineu64tou256(1:u64,2:u64,3:u64,4:u64)`                                         |
|           | `keccak256(p:u256, s:u256) -> v:u256`                                                  | 哈希计算                  | `let h := keccak256(0x40:u256, 0x20:u256)`                                                   |
---

我帮你整理成一份 **Yul 对象笔记**，包括语法说明、组成元素和示例说明：

---

## 对象说明

Yul 对象是 Yul 的顶层封装结构，可以包含代码、嵌套对象和数据。它通常用于描述合约的构造函数和运行时代码

**语法**:

```
顶层对象   = 'object' '{' 代码? ( 对象 | 数据 )* '}'
对象       = 'object' 字符串字面量 '{' 代码? ( 对象 | 数据 )* '}'
代码       = 'code' 代码块
数据       = 'data' 字符串字面量 十六进制字面量
十六进制字面量 = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
字符串字面量 = '"' ([^"\r\n\\] | '\\' .)* '"'
```

* **代码块**指的是 Yul 代码语法中的 `代码块`（前一章已说明）
* **对象**可嵌套，支持构建工厂模式或模块化合约
* **数据**可以通过内置函数 `datacopy` / `dataoffset` / `datasize` 在运行时访问



### 组成元素

| 元素     | 描述                         |
| ------ | -------------------------- |
| object | 顶层或嵌套对象，包含代码和/或子对象、数据      |
| code   | 代码块，通常是构造函数或运行时代码          |
| data   | 二进制数据，可通过字符串字面量命名，用 hex 定义 |
| 嵌套对象   | 内层对象可作为模块或工厂生成的合约          |



### 内置函数与数据访问

| 函数                         | 描述                     | 示例                                              |
| -------------------------- | ---------------------- | ----------------------------------------------- |
| `datasize("name")`         | 获取名为 `name` 的数据大小      | `let size = datasize("runtime")`                |
| `dataoffset("name")`       | 获取名为 `name` 的数据在内存中的偏移 | `let offset = dataoffset("runtime")`            |
| `datacopy(src, dest, len)` | 从 src 拷贝 len 字节到 dest  | `datacopy(dataoffset("runtime"), offset, size)` |
| `allocate(size)`           | 为数据分配内存                | `let offset = allocate(size)`                   |
| `return(offset, size)`     | 返回构造函数结果（runtime code） | `return(offset, size)`                          |
| `create(offset, size)`     | 在 EVM 上创建新合约           | `create(offset, add(size, 32))`                 |

**示例**:

```yul
object {
    code {
        let size = datasize("runtime")
        let offset = allocate(size)
        datacopy(dataoffset("runtime"), offset, size)
        return(offset, size)   // 构造函数返回运行时代码
    }

    data "Table2" hex"4123"  // 二进制数据

    object "runtime" {
        code {
            let size = datasize("Contract2")
            let offset = allocate(size)
            datacopy(dataoffset("Contract2"), offset, size)
            mstore(add(offset, size), 0x1234)
            create(offset, add(size, 32))  // 工厂模式创建 Contract2
        }

        object "Contract2" {
            code {
                // Contract2 构造函数代码
            }

            object "runtime" {
                code {
                    // Contract2 运行时代码
                }
             }

             data "Table1" hex"4123"  // Contract2 内部数据
        }
    }
}
```

* 顶层 `object` 包含构造函数代码 `code`、数据 `Table2` 和嵌套对象 `runtime`
* 嵌套对象 `runtime` 又包含另一个对象 `Contract2`，实现工厂模式生成合约
* 内置函数 `datacopy` 和 `dataoffset` 用于将数据复制到内存，为 `create` 提供参数
* `return` 用于构造函数返回运行时代码

---

## Solidity 内联 Yul
**示例**:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract InlineYulDemo {
    function add(uint256 a, uint256 b) external pure returns (uint256 result) {
        assembly {
            result := add(a, b)
            //支持所有 yul 语法
        }
    }
}
```

