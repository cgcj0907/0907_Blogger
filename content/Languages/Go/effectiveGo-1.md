---
title: "Effective Go I"
date: 2025-12-30
draft: false
tags: ["Languages", "Golang"]
ShowToc: true
---

## Formatting

Go 语言自带格式化工具
```bash
# 格式化文件并输出到标准输出
gofmt main.go

# 直接修改文件
gofmt -w main.go

# 格式化整个目录
gofmt -w ./...
```

## [Commentary](https://go.dev/doc/comment)

Go 支持使用 `//` 单行注释 和 `/* */` 多行注释


## Names

### Package names

Go 的包导入, 默认使用最后一层目录名作为名称, 即
```go
import "a/b/c" //使用时用 c
```

### Getters

Go 语言中不自带 getters/setters, 如需自定义, 建议 getters 直接以本体命名, 无需加 Get, 例如
```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

### Interface names
- 单方法接口命名
    - 用方法名 + `-er` 后缀命名，表示执行动作的角色。
    - 例如：Reader, Writer, Closer, Formatter

- 常用动作方法名
    - Read, Write, Close, Flush, String 等有固定签名和语义
    - 不要随意改名，避免混淆
    - 示例：
    ```go
    func (t MyType) String() string { ... } // 正确
    func (t MyType) ToString() string { ... } // 不推荐
    ```

### MixedCaps

Go 语言一般使用小驼峰或者大驼峰, 不建议使用下划线命名


## Semicolons

编写 Go 程序时无需加分号, 编译器会根据结尾字符自行加分号(ASI)

因为上述特性导致编写 `if` 等类似语句时左大括号不能另起一行

正确示例:
```go
if i < f() {
    g()
}
```

错误示例:
```go
if i < f()  // wrong!
{           // wrong!
    g()
}
```

## Control structures

### if

在 Go 语言条件语句无需 () 包裹

Go 还有一个特性与其他语言不同, 就是可以在条件语句中赋值, 例如
```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

### Redeclaration and reassignment

1. **部分重声明**

   ```go
   f, err := os.Open(name)  // 声明 f 和 err
   d, err := f.Stat()       // 声明 d，err 复用已有变量
   ```

   * 至少有一个新变量才能用 `:=`
   * 已存在的变量会被重新赋值

2. **作用域**

   * 变量必须在同一作用域
   * 外层作用域的变量不会被覆盖，而是新建局部变量
   * 函数参数和返回值与函数体共享作用域

3. **用途**

   * 常用于复用 `err` 变量
   * 避免重复声明，提高代码简洁性

### For

Go 语言中有类似与 C 语言中的 `For` 和 `while`, 但是没有 `do while`
```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

与 C 语言一样, 可以在 `for` 里定义
```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

具有类似于 Python 的 `range` 操作
```go
for key, value := range oldMap {
    newMap[key] = value
}
```

可以根据需要省略
```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```
```go
sum := 0
for _, value := range array {
    sum += value
}
```

### Switch

1. 基本特性

* 表达式可以是任意类型，也可以省略（相当于 `switch true`）
* case 从上到下匹配，遇到第一个匹配就执行
* 默认没有自动 fall-through，每个 case 执行完自动结束

```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```


2. 多值 case

* 用逗号分隔多个匹配值

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```


3. break 与标签 break

* **break**：退出当前 switch
* **带标签的 break**：退出外层循环或任意带标签的代码块

```go
Loop:
for n := 0; n < len(src); n += size {
    switch {
    case src[n] < sizeOne:
        if validateOnly {
            break
        }
        size = 1
        update(src[n])
    case src[n] < sizeTwo:
        if n+1 >= len(src) {
            err = errShortInput
            break Loop
        }
        if validateOnly {
            break
        }
        size = 2
        update(src[n] + src[n+1]<<shift)
    }
}
```

### Type Switch

一个 switch 语句也可以用于发现接口变量的动态类型。这种类型开关使用类型断言的语法，在括号内使用关键字 type。如果在表达式中声明了一个变量，该变量将在每个子句中具有对应的类型。在这种情况下，习惯上会重用变量名，实际上在每个 case 中声明了一个同名但类型不同的新变量

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("意外的类型 %T\n", t)     // %T 打印 t 的实际类型
case bool:
    fmt.Printf("布尔值 %t\n", t)         // t 的类型为 bool
case int:
    fmt.Printf("整数 %d\n", t)           // t 的类型为 int
case *bool:
    fmt.Printf("指向布尔值的指针 %t\n", *t) // t 的类型为 *bool
case *int:
    fmt.Printf("指向整数的指针 %d\n", *t)   // t 的类型为 *int
}
```

## Functions

### Multiple return values

Go 语言支持多返回值
在 C 语言中错误信息通常返回 `-1`, 而在 Go 中通常返回 error 类型字段

```go
func (file *File) Write(b []byte) (n int, err error)
```

### Named result parameters

Go 语言支持直接给返回值命名, 并在函数中使用, 例如
```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### Defer

被 defer 命名的函数将在执行它的函数返回前运行, 例如
```go
 // Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

> 注意 defer 遵循 LIFO (后进先出) 原则

## Data

### Allocation with new

Go 语言有两种分配原生语言, 内置函数 `new` 和 `make`

new(T) 会分配被 0 覆盖的空间, 不会初始化内部结构, 并返回指针类型(执行空间的地址), 例如
```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}

p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

在 Go 中，有些类型 **零值就可以直接使用**，不需要显式初始化或构造函数

* `bytes.Buffer`：

  * 零值就是一个空的、可用的缓冲区
  * 可以直接调用 `Write`、`Read` 等方法

* `sync.Mutex`：

  * 零值就是**未锁定状态**的互斥锁
  * 不需要 `Init` 或构造函数就可以直接使用 `Lock` / `Unlock`

而有些类型不能直接用：

* **Slice**：零值是 nil，不能直接 append，需 `make([]T, len)`
* **Map**：零值是 nil，不能直接赋值，需 `make(map[K]V)`
* **Channel**：零值是 nil，不能发送/接收，需 `make(chan T, N)`


### Constructors and composite literals

如果不想将结构体全部被 0 覆盖, 可以自定义创建函数, 例如
```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```

注意到, 在 Go 中，**可以安全地返回局部变量的地址**
和 C 不同，函数返回后该变量的内存不会失效

原因是：

* Go 会自动做 **逃逸分析**
* 如果变量的地址被返回，编译器会把它分配到堆上
* 生命周期会延续到不再被使用为止

Go 的数组、切片和 map 都可以用**带标签的复合字面量**来初始化

* 对数组、切片：标签是 **下标**
* 对 map：标签是 **key**
* 初始化时**不要求按顺序写**
* 只要 `Enone`、`Eio`、`Einval` 是不同的整数，结果就正确

```go
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

### Allocation with make

`make` 和 `new` 区别：

* **make(T, args)**

  * 只用于 slice、map、channel
  * 返回已初始化的值（T 本身），可直接使用
  * 初始化内部结构（slice 有指针、len、cap）

* **new(T)**

  * 分配零值内存，返回 `*T`
  * slice/map/channel 零值是 nil，不能直接用

示例：

```go
var p *[]int = new([]int)       // *p == nil
var v  []int = make([]int, 100) // 可直接使用
```

原则：**需要初始化内部结构的类型用 make，普通内存分配用 new**

### Arrays

Go 中数组特点：

* **数组是值类型**：赋值会拷贝所有元素，传函数也是拷贝
* **数组长度是类型的一部分**：如 `[10]int` 和 `[20]int` 类型不同
* **可以通过指针避免拷贝**：

```go
func Sum(a *[3]float64) float64 {
    var sum float64
    for _, v := range *a {
        sum += v
    }
    return sum
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)
```

* **注意**：这种指针方式在 Go 中不常用，推荐使用 slice

### Slices

Go 中 slice 特点：

* **slice 包装数组**，提供更灵活、方便的序列操作

* **slice 是引用类型**：赋值或传函数会共享同一底层数组，修改会影响原数组

* **长度和容量**：

  * `len(slice)` 返回当前长度
  * `cap(slice)` 返回最大长度（底层数组大小）
  * 可以通过 `slice = slice[:newLen]` 调整长度，只要不超过容量

* **读取数据**：

```go
n, err := f.Read(buf[0:32])
```

* **append 自行扩容**：

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l+len(data) > cap(slice) {
        newSlice := make([]byte, (l+len(data))*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[:l+len(data)]
    copy(slice[l:], data)
    return slice
}
```

* **注意**：slice 本身按值传递，必须返回新的 slice
* **内置 `append` 函数** 已封装此逻辑，可直接使用

#### Two-dimensional slices

Go 的数组和 slice 是一维的。要表示二维，可以用数组的数组或 slice 的 slice：

```go
type Transform [3][3]float64   // 3x3 数组
type LinesOfText [][]byte      // slice 的 slice
```

* 内层 slice 可以长度不同

```go
text := LinesOfText{
    []byte("Now is the time"),
    []byte("for all good gophers"),
    []byte("to bring some fun to the party."),
}
```

二维 slice 分配有两种方式：

1. 逐行分配（行可变长）

```go
picture := make([][]uint8, YSize)
for i := range picture {
    picture[i] = make([]uint8, XSize)
}
```

* 每行独立分配，长度可变
* 内存不连续
* 灵活，适合行长度不同

2. 一次性分配（固定行列）

```go
picture := make([][]uint8, YSize)
pixels := make([]uint8, XSize*YSize)
for i := range picture {
    picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

* 所有元素在一个连续数组
* 每行是 slice 视图
* 节省内存，访问快，但行长度固定

### Map

Go 的 map 特点：

* **键值对**：key 类型可以用 `==` 比较的类型，slice 不能作 key (因为 slice 没有实现 `==`)
* **引用类型**：传函数修改会影响原 map
* **初始化**：

```go
timeZone := map[string]int{
    "UTC": 0*60*60,
    "EST": -5*60*60,
}
```

* **访问和赋值**：`offset := timeZone["EST"]`
* **不存在的 key** 返回值类型零值，如 int 返回 0
* **实现集合**：value 用 bool

```go
attended := map[string]bool{"Ann": true}
if attended[person] { ... }
```

* **判断 key 是否存在**（comma ok）：

```go
seconds, ok := timeZone[tz]  // ok 为 true 表示存在
```

* **只关心存在性**：

```go
_, present := timeZone[tz]
```

* **删除元素**：

```go
delete(timeZone, "PDT")
```

### Printing

Go 的格式化打印采用类似 C 语言 `printf` 的风格，但功能更丰富。相关函数位于 `fmt` 包中，函数名首字母大写：`fmt.Printf`、`fmt.Fprintf`、`fmt.Sprintf` 等。

#### 三类打印函数

1. 格式化打印函数
- `Printf` - 输出到标准输出
- `Fprintf` - 输出到指定的 `io.Writer`
- `Sprintf` - 返回格式化后的字符串

```go
fmt.Printf("Hello %d\n", 23)  // 需要格式字符串
```

2. 非格式化打印函数
- `Print` - 自动格式，参数间无分隔符（除非两边都不是字符串）
- `Println` - 自动格式，参数间加空格，末尾加换行符

```go
fmt.Println("Hello", 23)  // 自动格式，输出 "Hello 23\n"
```

3. 字符串生成函数
- `Sprint` - 返回自动格式化的字符串
- `Sprintln` - 返回带空格和换行的字符串

```go
result := fmt.Sprint("Hello ", 23)
```

#### 格式说明符的特点

与 C 语言的区别
1. 数字格式符（如 `%d`）不需要符号或大小标志
2. 打印例程根据参数类型决定显示属性

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
// 输出: 18446744073709551615 ffffffffffffffff; -1 -1
```

#### 通用格式符 `%v`

基本用法
- `%v` - 默认格式（相当于 `Print` 的输出）
- 可打印任何值，包括数组、切片、结构体和映射

```go
fmt.Printf("%v\n", timeZone)  // 等同于 fmt.Println(timeZone)
```

映射(Map)打印特性
- `Printf` 系列函数按键的字典顺序排序输出

#### 高级格式选项

结构体打印
- `%+v` - 显示字段名
- `%#v` - 完整的 Go 语法表示

```go
type T struct {
    a int
    b float64
    c string
}
t := &T{7, -2.35, "abc\tdef"}

fmt.Printf("%v\n", t)   // &{7 -2.35 abc   def}
fmt.Printf("%+v\n", t)  // &{a:7 b:-2.35 c:abc     def}
fmt.Printf("%#v\n", t)  // &main.T{a:7, b:-2.35, c:"abc\tdef"}
```

其他格式符
- `%q` - 带引号的字符串（可用于字符串、字节切片）
- `%#q` - 尽可能使用反引号
- `%x` - 十六进制格式
- `% x` - 十六进制，字节间加空格
- `%T` - 打印类型

```go
fmt.Printf("%T\n", timeZone)  // 输出: map[string]int
```

#### 自定义类型的格式化

* String() 方法
为自定义类型实现 `String() string` 方法即可控制默认格式

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)  // 输出: 7/-2.35/"abc\tdef"
```

**注意事项**
1. **避免递归调用**：在 `String()` 方法中调用 `Sprintf` 时要小心
2. **错误示例**：
```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m)  // 错误：无限递归
}
```
3. **正确做法**：转换为基本类型
```go
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m))  // 正确
}
```

#### 可变参数传递

可变参数函数
- 使用 `...interface{}` 表示任意数量、任意类型的参数
- 函数内部 `v` 的类型为 `[]interface{}`
- 传递给其他可变参数函数时，需要使用 `...` 展开

```go
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // 注意：v... 展开参数
}
```

特定类型的可变参数
```go
func Min(a ...int) int {
    min := int(^uint(0) >> 1)  // 最大整数值
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}
```


### Append

Go 的 `append` 是内置函数，用于 **把元素添加到 slice 末尾**，并返回可能改变后的 slice。

---

#### 语法

```go
func append(slice []T, elements ...T) []T
```

* `T` 是任意类型（编译器支持，用户无法写泛型 T）
* 第二个参数是可变参数，可以传多个元素


#### 示例

```go
x := []int{1,2,3}
x = append(x, 4,5,6)  // 添加单独元素
fmt.Println(x)         // [1 2 3 4 5 6]
```

* 如果要 **把一个 slice 添加到另一个 slice**，必须在调用时加 `...` 展开：

```go
y := []int{4,5,6}
x = append(x, y...)    // 展开 y 的元素
fmt.Println(x)         // [1 2 3 4 5 6]
```

* **不加 `...`** 会报错，因为类型不匹配，`y` 是 `[]int`，而 `append` 需要 `int`。

