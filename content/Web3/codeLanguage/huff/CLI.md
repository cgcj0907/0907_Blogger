---
title: "Huff CLI"
date: 2025-05-16
draft: false
tags: ["Web3", "lanuage", "Huff"]
showToc: true
---

# 简介
虽然多数时候您会在Foundry项目中使用foundry-huff库来编译Huff合约，但编译器自带的CLI提供了一些额外的配置选项和有用的工具

## 选项
```bash
`huffc 0.3.0`
用纯Rust构建的Huff语言编译器

使用格式：
`huffc [选项] [路径] [子命令]`

参数：
   `<路径>` 要编译的合约（一个或多个）

选项：
   `-a`, `--artifacts` 是否生成产物文件
   `-b`, `--bytecode` 生成并记录字节码
   `-c`, `--constants <常量>...` 为编译环境覆盖/设置常量
   `-d`, `--output-directory <输出目录>` 输出目录 [默认: ./artifacts]
   `-g`, `--interface [<接口>...]` 为Huff产物文件生成Solidity接口
   `-h`, `--help` 打印帮助信息
   `-i`, `--inputs <输入参数>...` 构造函数输入参数
   `-n`, `--interactive` 交互式输入构造函数参数
   `-o`, `--output <输出>` 输出文件路径
   `-r`, `--bin-runtime` 生成并记录运行时字节码
   `-s`, `--source-path <源路径>` 合约的源路径 [默认: ./contracts]
   `-v`, `--verbose` 详细输出
   `-V`, `--version` 打印版本信息
   `-z`, `--optimize` 优化编译 [开发中]

子命令：
   `help` 打印此信息或给定子命令的帮助信息
   `test` 测试子命令
```
### `-a` 生成产物文件
传递 `-a` 标志将在 `./artifacts` 目录或 `-d` 标志指定的位置生成产物JSON文件。该JSON文件包含以下信息：
*   File（文件名）
*   Path（路径）
*   Source（源代码）
*   Dependencies（依赖项）
*   Bytecode（字节码）
*   Runtime Bytecode（运行时字节码）
*   Contract ABI（合约ABI）

**示例：**
```bash
huffc ./src/ERC20.huff -a
```

### `-b` 生成字节码
传递 `-b` 标志将指示编译器在控制台打印编译过程中生成的字节码

**示例：**
```bash
huffc ./src/ERC20.huff -b
```

### `-c` 设置常量
**参数：** `[常量]`

传递 `-c` 标志允许您为当前编译环境覆盖和设置常量。字面量必须以 `0x` 格式提供，且长度需 `<= 32` 字节

**示例：**
```bash
huffc ./Test.huff -c MY_CONST=0x01 MY_OTHER_CONST=0xa57b
```

### `-d` 输出目录
**参数：** `<输出目录>`，**默认：** `./artifacts`

传递 `-d` 标志允许您指定产物JSON文件的导出目录

**示例：**
```bash
huffc ./src/ERC20.huff -d ./my_artifacts
```

### `-g` 生成接口
传递 `-g` 标志将为提供的Huff合约生成一个Solidity接口。该接口是根据合约内的函数和事件定义生成的

生成的Solidity文件将始终命名为 `I<HUFF文件名>.sol`，并保存在与Huff合约相同的目录中。

**示例：**
```bash
huffc ./src/ERC20.huff -g
```

### `-i` 输入构造函数参数
**参数：** `[构造函数参数]`

传递 `-i` 标志允许您为正在编译的合约设置构造函数参数。所有输入参数应以逗号分隔。如果您希望以交互方式输入构造函数参数，请使用 `-n` 标志

**示例（假设 ERC20.huff 的构造函数接收一个字符串和一个 uint）：**
```bash
huffc ./src/ERC20.huff -i "TestToken", 18
```

### `-n` 交互式输入
传递 `-n` 标志允许您通过CLI交互式输入构造函数参数，而不是通过 `-i` 标志

**示例：**
```bash
huffc ./src/ERC20.huff -n
```

### `-o` 输出文件
**参数：** `<文件路径>`

传递 `-o` 标志允许您将产物导出到特定文件，而不是一个文件夹

**示例：**
```bash
huffc ./src/ERC20.huff -o ./artifact.json
```

### `-s` 源代码路径
**参数：** `<合约文件夹>`，**默认：** `./contracts`

传递 `-s` 标志允许您更改编译器扫描Huff合约的目录

**示例：**
```bash
huffc -s ./src/
```

### `-r` 生成运行时字节码
传递 `-r` 标志将指示编译器打印已编译合约的运行时字节码

### `-v` 详细输出
传递 `-v` 标志将指示编译器在编译过程中打印详细输出。此输出对于调试合约和编译器错误非常有用

**示例：**
```bash
huffc ./src/ERC20.huff -v
```

### `-z` 优化
编译器**尚未实现**此功能

## 子命令

### `test`
**格式：** `huffc ./path/to/Contract.huff test [-f <list|table|json>] [-m <测试名称>]`
`test` 子命令是运行Huff合约内测试的入口

**可选标志：**
*   `-f` 或 `--format`：将测试报告格式化为列表、表格或JSON
*   `-m` 或 `--match`：运行与此标志传递的名称匹配的特定测试