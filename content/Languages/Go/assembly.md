---
title: "初识汇编-Golang"
date: 2025-12-29
draft: false
tags: ["Languages", "Golang", "assembly"]
ShowToc: true
---

## 伪汇编
![](/Languages/Go/go_assembly.webp)
**Go 汇编是抽象中间层**
* Go 编译器输出的是一种抽象的、可移植的汇编形式
* 这种汇编不直接对应真实硬件指令
* Go 汇编器将这个"伪汇编"转换为具体硬件的机器指令

**设计目标:易于移植**
* 主要优势：简化 Go 语言向新架构的移植
* 只需要为新硬件实现汇编器后端，无需重写整个编译器
* 这是 Go 能快速支持多种架构的关键设计

## 简单示例
代码示例
```go
package main
//go:noinline
func add(a, b int32) (int32, bool) { 	
	return a + b, true 
}

func main() { add(10, 32) }	
```
> //go:noinline 即编译时采取无内联的形式, 有无内联的区别可以参照下图
![](/Languages/Go/inline_noinline.webp)

编译示例
> 以 WSL Ubuntu 64位 为例
```bash
# -S 打印汇编指令信息 -N 关闭编译器优化
# 读者也可以不使用 -N 自行观察两种模式下编译的产物差异
w@LAPTOP-IDKREHQV:~/go-test$ go tool compile -S -N main.go

# add 函数编译指令
main.add STEXT nosplit size=51 args=0x8 locals=0x10 funcid=0x0 align=0x0
        0x0000 00000 (/home/w/go-test/main.go:3)        TEXT    main.add(SB), NOSPLIT|ABIInternal, $16-8
        0x0000 00000 (/home/w/go-test/main.go:3)        PUSHQ   BP
        0x0001 00001 (/home/w/go-test/main.go:3)        MOVQ    SP, BP
        0x0004 00004 (/home/w/go-test/main.go:3)        SUBQ    $8, SP
        0x0008 00008 (/home/w/go-test/main.go:3)        FUNCDATA        $0, gclocals·g5+hNtRBP6YXNjfog7aZjQ==(SB)
        0x0008 00008 (/home/w/go-test/main.go:3)        FUNCDATA        $1, gclocals·g5+hNtRBP6YXNjfog7aZjQ==(SB)
        0x0008 00008 (/home/w/go-test/main.go:3)        FUNCDATA        $5, main.add.arginfo1(SB)
        0x0008 00008 (/home/w/go-test/main.go:3)        MOVL    AX, main.a+24(SP)
        0x000c 00012 (/home/w/go-test/main.go:3)        MOVL    BX, main.b+28(SP)
        0x0010 00016 (/home/w/go-test/main.go:3)        MOVL    $0, main.~r0+4(SP)
        0x0018 00024 (/home/w/go-test/main.go:3)        MOVB    $0, main.~r1+3(SP)
        0x001d 00029 (/home/w/go-test/main.go:4)        ADDL    BX, AX
        0x001f 00031 (/home/w/go-test/main.go:4)        MOVL    AX, main.~r0+4(SP)
        0x0023 00035 (/home/w/go-test/main.go:4)        MOVB    $1, main.~r1+3(SP)
        0x0028 00040 (/home/w/go-test/main.go:4)        MOVL    $1, BX
        0x002d 00045 (/home/w/go-test/main.go:4)        ADDQ    $8, SP
        0x0031 00049 (/home/w/go-test/main.go:4)        POPQ    BP
        0x0032 00050 (/home/w/go-test/main.go:4)        RET


# main 函数编译指令
main.main STEXT size=42 args=0x0 locals=0x10 funcid=0x0 align=0x0
        0x0000 00000 (/home/w/go-test/main.go:7)        TEXT    main.main(SB), ABIInternal, $16-0
        0x0000 00000 (/home/w/go-test/main.go:7)        CMPQ    SP, 16(R14)
        0x0004 00004 (/home/w/go-test/main.go:7)        PCDATA  $0, $-2
        0x0004 00004 (/home/w/go-test/main.go:7)        JLS     35
        0x0006 00006 (/home/w/go-test/main.go:7)        PCDATA  $0, $-1
        0x0006 00006 (/home/w/go-test/main.go:7)        PUSHQ   BP
        0x0007 00007 (/home/w/go-test/main.go:7)        MOVQ    SP, BP
        0x000a 00010 (/home/w/go-test/main.go:7)        SUBQ    $8, SP
        0x000e 00014 (/home/w/go-test/main.go:7)        FUNCDATA        $0, gclocals·g5+hNtRBP6YXNjfog7aZjQ==(SB)
        0x000e 00014 (/home/w/go-test/main.go:7)        FUNCDATA        $1, gclocals·g5+hNtRBP6YXNjfog7aZjQ==(SB)
        0x000e 00014 (/home/w/go-test/main.go:7)        MOVL    $10, AX
        0x0013 00019 (/home/w/go-test/main.go:7)        MOVL    $32, BX
        0x0018 00024 (/home/w/go-test/main.go:7)        PCDATA  $1, $0
        0x0018 00024 (/home/w/go-test/main.go:7)        CALL    main.add(SB)
        0x001d 00029 (/home/w/go-test/main.go:7)        ADDQ    $8, SP
        0x0021 00033 (/home/w/go-test/main.go:7)        POPQ    BP
        0x0022 00034 (/home/w/go-test/main.go:7)        RET
        0x0023 00035 (/home/w/go-test/main.go:7)        NOP
        0x0023 00035 (/home/w/go-test/main.go:7)        PCDATA  $1, $-1
        0x0023 00035 (/home/w/go-test/main.go:7)        PCDATA  $0, $-2
        0x0023 00035 (/home/w/go-test/main.go:7)        CALL    runtime.morestack_noctxt(SB)
        0x0028 00040 (/home/w/go-test/main.go:7)        PCDATA  $0, $-1
        0x0028 00040 (/home/w/go-test/main.go:7)        JMP     0                               .....
```
### add 函数编译解析
1. **标头字段解释**
```bash
TEXT    main.add(SB), NOSPLIT|ABIInternal, $16-8
```
* NOSPLIT: 表示不用进行栈溢出检测, 通常存在于没有本地变量的函数, 无需额外栈帧分配
* $16-8: `16` 表示分配 16 字节大小的栈帧(之后会解释 16 从哪里来), `8` 表示参数大小位 8 字节
2. **golang 栈特性**
在 golang 中栈的分配大多数情况下不通过 PUSH/POP, 而是通过改变 SP 寄存器的指针指向,即:
```bash
SUBQ    $8, SP # 将 SP 指针下移, 分配额外栈空间
ADDQ    $8, SP # 函数运行完成后, 将 SP 指针上移, 回收先前分配的栈空间
```
3. **栈空间分布**
了解了 golang 栈特性, 现在我们来分析示例代码 add 函数在运行时的栈空间分布
* 在 golang 编译指令中数据通过相对 SP 的位置来访问, 例如 `main.a+32(SP)` 即 a 在当前 SP 指针 上方 32 字节处
* `PUSHQ BP`, 即将 BP 压入栈顶, 此时 SP - 8, 再加上 `SUBQ $8, SP`, 刚好给 add 函数分配 16 字节栈帧, 对应了 `$16`
* `CALL main.add(SB)`, 即调用 add 函数, 同时会把返回地址压入栈顶, 此时 SP - 8

综合以下代码分析
```bash
PUSHQ   BP
MOVQ    SP, BP
SUBQ    $8, SP
...
MOVL    AX, main.a+24(SP)
MOVL    BX, main.b+28(SP)
MOVL    $0, main.~r0+4(SP)
MOVB    $0, main.~r1+3(SP)
ADDL    BX, AX
MOVL    AX, main.~r0+4(SP)
MOVB    $1, main.~r1+3(SP)
...
```
可得如下栈空间分布试图



### main 函数编译解析