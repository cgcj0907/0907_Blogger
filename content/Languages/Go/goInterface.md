---
title: "从汇编看世界-Go Interface"
date: 2025-12-31
draft: false
tags: ["Languages", "Golang", "assembly"]
ShowToc: true
---

# 本章示例代码

> 运行环境 WSL Ubuntu 24.04 LTS 64位

```go
// 文件名 main.go
package main

type Mather interface {
    Add(a, b int32) int32
    Sub(a, b int64) int64
}

type Adder struct{ id int32 }
//go:noinline
func (adder Adder) Add(a, b int32) int32 { return a + b }
//go:noinline
func (adder Adder) Sub(a, b int64) int64 { return a - b }

func main() {
    m := Mather(Adder{id: 6754})

    // This call just makes sure that the interface is actually used.
    // Without this call, the linker would see that the interface defined above
    // is in fact never used, and thus would optimize it out of the final
    // executable.
    m.Add(10, 32)

}
```

## 从源码看 Interface 结构组成

### [iface](https://github.com/golang/go/blob/bf86aec25972f3a100c3aa58a6abcbcc35bdea49/src/runtime/runtime2.go#L143-L146)

> iface 是在运行过程中用来表示 interface 的结构体, 如下
```go
// 在 64 位计算机中, 该结构体大小为 16 字节, 本质上就是两个指向地址的指针
type iface struct { 
    tab  *itab
    data unsafe.Pointer
}
```
* `tab`:  保存着 itab 对象的地址，该对象包含了描述接口类型以及接口所指向的数据类型的数据结构
* `data`: 是一个原始指针（即不安全的指针），指向接口所持有的实际值

#### [itab](https://github.com/golang/go/blob/bf86aec25972f3a100c3aa58a6abcbcc35bdea49/src/runtime/runtime2.go#L648-L658)
```go
type itab struct { 
    // 在 64 位计算机上, 为 40 字节
    inter *interfacetype
    _type *_type
    hash  uint32 // _type 的哈希值
    _     [4]byte
    fun   [1]uintptr // 大小可变. 如果 fun[0] == 0，表示该 _type 没有实现对应的接口
}
```

## 从汇编看 Interface 创建过程
> 在了解完 Interface 的基本结构 iface 后, 读者先自行执行编译命令得到编译产物

> 编译命令 go tool compile -N -S main.go (会出现 unlinkable 字段) 或者 go build -gcflags='-N -S' main.go 

> 下面我不会列举完整的编译产物

> 细心的读者可能会发现编译产物中 main 函数没有 add 函数的调用, 这是因为编译器优化问题, 如果读者想体现 add 函数的调用, 则需自行增加函数返回结果的使用逻辑来防止编译器优化

总体步骤分为 3 步
1. 创建接收者(receiver) m
> 该步骤无法直接看出, 因为接收者 m 此时并不在栈中创建

> 执行命令 go tool compile -N -S main.go | grep -A 2 '^main..stmp_0'
```
main..stmp_0 SRODATA static size=4
        0x0000 62 1a 00 00                                      b...
```
按小端序解析: 0x1a62 = 6754, 刚好对应
2. 创建 itab
> 执行命令 go tool compile -N -S main.go | grep -A 7 '^go:itab.<unlinkable>.Adder,<unlinkable>.Mather'
```
go:itab.<unlinkable>.Adder,<unlinkable>.Mather SRODATA dupok size=40
        0x0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
        0x0010 9b b6 43 05 00 00 00 00 00 00 00 00 00 00 00 00  ..C.............
        0x0020 00 00 00 00 00 00 00 00                          ........
        rel 0+8 t=R_ADDR type:<unlinkable>.Mather+0 
        rel 8+8 t=R_ADDR type:<unlinkable>.Adder+0
        rel 24+8 t=RelocType(-32767) <unlinkable>.(*Adder).Add+0
        rel 32+8 t=RelocType(-32767) <unlinkable>.(*Adder).Sub+0
```
读者可自行按照 itab 结构体 对应
```go
type itab struct { 
    inter *interfacetype // offset 0x00 ($00)
    _type *_type	 // offset 0x08 ($08)
    hash  uint32	 // offset 0x10 ($16)
    _     [4]byte	 // offset 0x14 ($20)
    fun   [1]uintptr	 // offset 0x18 ($24)
			 // offset 0x20 ($32)
}
``` 
可能有细心的读者发现了有一串二进制数大多数都为0
```
        0x0000 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
        0x0010 9b b6 43 05 00 00 00 00 00 00 00 00 00 00 00 00  ..C.............
        0x0020 00 00 00 00 00 00 00 00                          ........
```
唯一不为 0 的部分在 0x0010 的前四个字节 `9b b6 43 05`, 即对应 hash 字段, 那么剩余的部分为什么为 0 呢, 其实不是凭空消失只不过是目前没有找到对应的地址而已, 接下来我将带领读者找到真实的数据
* 首先, 先执行命令 go build -o main.bin main.go
    * 执行 readelf -St -W main.bin 可查看当前 ELF 文件的 section headers
* 执行以下命令分别获取 `.rodata` 的真实文件偏移地址和虚拟地址`
```bash
readelf -St -W main.bin | \
  grep -A 1 .rodata | \
  tail -n +2 | \
  awk '{print "ibase=16;"toupper($3)}' | \
  bc
458752 //真实地址

readelf -St -W main.bin | \
  grep -A 1 .rodata | \
  tail -n +2 | \
  awk '{print "ibase=16;"toupper($2)}' | \
  bc
4653056 //虚拟地址
```
* 执行下面命令得到 itab 的虚拟地址
```bash
objdump -t -j .rodata main.bin | \
  grep "go.itab.main.Adder,main.Mather" | \
  awk '{print "ibase=16;"toupper($1)}' | \
  bc
4861336
```
* 计算真实 itab 便宜地址
先列举之前得到的数据
```
.rodata offset: 458752
.rodata VMA: 4653056

itab VMA: 0x475140 == 4861336
itab size: 0x24 = 40
```
根据公式 symbol_file_offset = symbol_vma - section_vma + section_offset
```
4861336 - 4653056 + 458752
= 208280 + 458752
= 667032
```
* 查询数据
```bash
dd if=main.bin of=/dev/stdout bs=1 count=40 skip=667032 2>/dev/null | hexdump -C

00000000  00 90 47 00 00 00 00 00  20 bd 47 00 00 00 00 00  |..G..... .G.....|
00000010  ae 2f 32 ab 00 00 00 00  80 ff 40 00 00 00 00 00  |./2.......@.....|
00000020  80 ff 40 00 00 00 00 00                           |..@.....|
00000028
```
> 细心的读者可能会发现此时的 hash 字段和之前的不一致, 这是因为前面是用 go tool compile 编译, 而现在是用 go build, 读者可自行用 go build 的编译产物对照, 发现数据一致

在编译指令中, 就是把创建好的 itab 放入栈中
```
        0x000e 00014 (/home/w/go-test/main.go:15)       LEAQ    go:itab.main.Adder,main.Mather(SB), DX
        0x0015 00021 (/home/w/go-test/main.go:15)       MOVQ    DX, main.m+24(SP)
```

3. 创建  data
在编译指令中就是将第一步创建的接收者实体地址放入栈中
```
        0x001a 00026 (/home/w/go-test/main.go:15)       LEAQ    main..stmp_0(SB), DX
        0x0021 00033 (/home/w/go-test/main.go:15)       MOVQ    DX, main.m+32(SP)
```


