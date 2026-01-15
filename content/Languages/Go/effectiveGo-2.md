---
title: "Effective Go II"
date: 2026-01-05
draft: false
tags: ["Languages", "Golang"]
ShowToc: true
---

## Initialization

### Constants

Go语言中的常量是不可变的。它们在编译时创建（即使在函数中定义为局部常量），并且只能是数字、字符（rune）、字符串或布尔值。由于编译时的限制，定义常量的表达式必须是编译时可计算的常量表达式。例如，`1<<3` 是常量表达式，而 `math.Sin(math.Pi/4)` 不是，因为对 `math.Sin` 的函数调用需要在运行时进行。

在Go中，使用 `iota` 枚举器来创建枚举常量。由于 `iota` 可以成为表达式的一部分，并且表达式可以隐式重复，因此很容易构建复杂的值集合。

```go
type ByteSize float64

const (
    _           = iota // 通过赋值给空标识符忽略第一个值
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

能够为任何用户定义的类型（如 `ByteSize`）附加 `String` 这类方法，使得任意值可以自动格式化打印。虽然这种方法最常用于结构体，但对于标量类型（如浮点类型）也同样有用

```go
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```

表达式 `YB` 打印为 `1.00YB`，而 `ByteSize(1e13)` 打印为 `9.09TB`。

这里使用 `Sprintf` 来实现 `ByteSize` 的 `String` 方法是安全的（避免无限递归），不是因为类型转换，而是因为它使用 `%f` 调用 `Sprintf`，这不是字符串格式：`Sprintf` 仅在需要字符串时才会调用 `String` 方法，而 `%f` 需要的是浮点数值


### Variables

变量的初始化方式与常量类似，但其初始化表达式可以是在运行时计算的任意表达式。

```go
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

### The init function

每个源文件都可以定义自己的无参 `init` 函数，以设置所需的任何状态（实际上每个文件可以有多个 `init` 函数）。`init` 函数会在包中所有变量声明完成初始化之后被调用，而这些初始化仅在所有导入的包都初始化完毕后才执行

除了用于无法用声明表达的初始化操作外，`init` 函数的常见用途是在程序真正开始执行前，验证或修复程序状态的正确性

```go
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath 可能被命令行中的 --gopath 标志覆盖
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

## Methods

### Pointers vs. Values

如我们之前用 `ByteSize` 看到的，**方法可以定义在任何命名类型上**（指针或接口类型除外）；接收器不一定必须是结构体

在上文关于切片的讨论中，我们编写了一个 `Append` 函数。我们可以将其定义为切片的方法。为此，我们首先声明一个命名类型来绑定该方法，然后让该方法的接收器成为该类型的值

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // 函数体与之前定义的 Append 函数完全相同
}
```

但这仍然需要方法返回更新后的切片。我们可以通过重新定义方法，使用指向 `ByteSlice` 的指针作为接收器，从而消除这种不便——这样方法就能直接覆盖调用者的切片

```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // 函数体同上，但无需返回
    *p = slice
}
```

实际上，我们还可以做得更好。如果我们将函数修改得看起来像标准的 `Write` 方法：

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // 同上
    *p = slice
    return len(data), nil
}
```

那么 `*ByteSlice` 类型就满足标准接口 `io.Writer`，这非常方便。例如，我们可以向其中写入内容：

```go
var b ByteSlice
fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

我们传递了 `ByteSlice` 的地址，因为只有 `*ByteSlice` 满足 `io.Writer`

**关于接收器的指针与值的规则是：**
- 值方法可以在指针和值上调用
- 但指针方法只能在指针上调用

这个规则的出现是因为指针方法可以修改接收器；在值上调用指针方法会导致该方法接收值的副本，因此任何修改都会被丢弃。因此，语言禁止这种错误。不过有一个便利的例外：**当值是可寻址的时，语言会自动插入地址操作符来处理在值上调用指针方法的常见情况**。在我们的例子中，变量 `b` 是可寻址的，因此我们可以直接用 `b.Write` 调用其 `Write` 方法。编译器会将其重写为 `(&b).Write`

完整示例:
```go
package main

import (
	"bytes"
	"fmt"
	"io"
)

type T struct {
	n int
}

/************** 方法定义 **************/

// 值接收器方法
func (t T) ValueMethod() {
	fmt.Println("ValueMethod, n =", t.n)
}

// 指针接收器方法（会修改接收器）
func (t *T) PointerMethod() {
	t.n++
	fmt.Println("PointerMethod, n =", t.n)
}

/************** 主函数 **************/

func main() {

	// ---------- 1. 普通变量（可寻址） ----------
	var a T

	a.ValueMethod()   // ✅ 值方法：值上调用
	a.PointerMethod() // ✅ 自动取地址 -> (&a).PointerMethod()

	fmt.Println("a after PointerMethod:", a.n)

	// ---------- 2. 指针 ----------
	p := &T{}

	p.ValueMethod()   // ✅ 值方法：指针上调用（自动解引用）
	p.PointerMethod() // ✅ 指针方法：指针上调用

	// ---------- 3. 字面量（不可寻址） ----------
	T{}.ValueMethod()   // ✅ 值方法
	// T{}.PointerMethod() // ❌ 编译错误：不可寻址，不能自动取地址

	// ---------- 4. 函数返回值（不可寻址） ----------
	getT().ValueMethod()   // ✅
	// getT().PointerMethod() // ❌ 编译错误：不可寻址

	// ---------- 5. map 元素（不可寻址） ----------
	m := map[string]T{"x": {n: 10}}

	m["x"].ValueMethod()   // ✅
	// m["x"].PointerMethod() // ❌ map 元素不可寻址

	// ---------- 6. 接口实现检查 ----------
	var _ io.Writer = (*bytes.Buffer)(nil) // ✅ bytes.Buffer 用指针实现接口

	// ---------- 7. bytes.Buffer 的核心思想 ----------
	var b bytes.Buffer // 可寻址

	b.Write([]byte("hello")) // ✅ 自动变成 (&b).Write
	fmt.Println(b.String())
}

/************** 辅助函数 **************/

func getT() T {
	return T{n: 100}
}
```

顺便提一下，在字节切片上使用 `Write` 的想法是 `bytes.Buffer` 实现的核心

## Interfaces and other types

### Interfaces

Go中的接口提供了一种指定对象行为的方式：如果某个对象能做这件事，那么它就能用在这里。我们已经见过几个简单的例子：自定义打印机可以通过`String`方法实现，而`Fprintf`可以向任何有`Write`方法的对象输出。在Go代码中，只有一两个方法的接口很常见，通常根据方法命名，比如实现了`Write`的`io.Writer`

一个类型可以实现多个接口。例如，一个集合如果实现了`sort.Interface`（包含`Len()`、`Less(i, j int) bool`和`Swap(i, j int)`方法），就可以用sort包中的例程排序，同时还可以有自定义格式化器。在这个刻意设计的例子中，`Sequence`同时满足两者：

```go
type Sequence []int

// sort.Interface要求的方法
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Copy返回Sequence的副本
func (s Sequence) Copy() Sequence {
    copy := make(Sequence, 0, len(s))
    return append(copy, s...)
}

// 打印方法 - 在打印前排序元素
func (s Sequence) String() string {
    s = s.Copy() // 创建副本，不覆盖参数
    sort.Sort(s)
    str := "["
    for i, elem := range s { // 循环为O(N²)，下个例子会修复
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```

### Conversions

`Sequence`的`String`方法重复了`Sprint`已经为切片做的工作。（复杂度还是O(N²)，这很糟糕。）如果我们能在调用`Sprint`前将`Sequence`转换为普通的`[]int`，就可以共享工作（还能加快速度）：

```go
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

这个方法是另一种从`String`方法安全调用`Sprintf`的转换技术。因为如果我们忽略类型名，这两种类型（`Sequence`和`[]int`）是相同的，所以它们之间的转换是合法的。这种转换不会创建新值，只是暂时让现有值表现为新类型。（还有其他合法的转换，比如从整数到浮点数，那些会创建新值。）

在Go程序中，转换表达式的类型以访问不同的方法集是一种惯用法。例如，我们可以使用现有的`sort.IntSlice`类型将整个例子简化为：

```go
type Sequence []int

// 打印方法 - 在打印前排序元素
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

现在，我们不再让`Sequence`实现多个接口（排序和打印），而是利用数据项可以转换为多种类型（`Sequence`、`sort.IntSlice`和`[]int`）的能力，每种类型完成部分工作。这在实际中不太常见，但可能很有效。

### Interface conversions and type assertions

类型开关（type switch）是转换的一种形式：它接受一个接口，对于switch中的每个case，在某种意义上将其转换为该case的类型。下面是`fmt.Printf`底层代码使用类型开关将值转换为字符串的简化版本。如果它已经是字符串，我们想要接口持有的实际字符串值；如果它有`String`方法，我们想要调用该方法的结果：

```go
type Stringer interface {
    String() string
}

var value interface{} // 调用者提供的值
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

第一个case找到具体值；第二个将接口转换为另一个接口。这样混合类型是完全没问题的。

如果我们只关心一种类型呢？如果我们知道值包含字符串，只想提取它？单case的类型开关可以做到，但类型断言也可以。类型断言接受一个接口值，从中提取指定显式类型的值。语法借用了开启类型开关的从句，但使用显式类型而不是`type`关键字：

```go
value.(typeName)
```

结果是一个具有静态类型`typeName`的新值。该类型必须是接口持有的具体类型，或者是值可以转换到的第二个接口类型。要提取我们知道在值中的字符串，可以写：

```go
str := value.(string)
```

但如果值不包含字符串，程序将在运行时崩溃。为了防止这种情况，使用"逗号, ok"惯用法来安全测试值是否为字符串：

```go
str, ok := value.(string)
if ok {
    fmt.Printf("字符串值为: %q\n", str)
} else {
    fmt.Printf("值不是字符串\n")
}
```

如果类型断言失败，`str`仍然存在且类型为`string`，但将具有零值——空字符串。

作为这种能力的说明，这里有一个if-else语句，等价于本节开头的类型开关：

```go
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
```

### Interfaces and methods

既然几乎任何类型都可以附加方法，那么几乎任何类型都能满足接口。`http` 包中的 `Handler` 接口就是一个很好的例子，任何实现了 `Handler` 的对象都可以处理 HTTP 请求

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

`ResponseWriter` 本身也是一个接口，提供了返回响应给客户端所需的方法，这些方法包括标准的 `Write` 方法，因此 `http.ResponseWriter` 可以用在任何需要 `io.Writer` 的地方。`Request` 是一个结构体，包含了解析后的客户端请求信息

为了简洁，我们忽略 POST 请求，假设 HTTP 请求总是 GET，这个简化不影响处理程序的设置方式。下面是一个简单的页面访问计数器的实现：

```go
// 简单的计数器服务器
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

（延续我们的主题，注意 `Fprintf` 如何输出到 `http.ResponseWriter`）在实际服务器中，对 `ctr.n` 的访问需要防止并发访问，可以参考 `sync` 和 `atomic` 包

如何将这样的服务器附加到 URL 树的节点上：

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

但为什么要将 `Counter` 设为结构体？一个整数就足够了（接收器需要是指针，这样增加操作对调用者可见）：

```go
// 更简单的计数器服务器
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

如果你的程序需要知道页面被访问的内部状态怎么办？将一个通道绑定到网页：

```go
// 在每次访问时发送通知的通道
//（可能需要通道有缓冲）
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

最后，假设我们想在 `/args` 页面上显示启动服务器时的参数。写一个打印参数的函数很容易：

```go
func ArgServer() {
    fmt.Println(os.Args)
}
```

如何将其转换为 HTTP 服务器？我们可以让 `ArgServer` 成为某个类型的方法而忽略其值，但有更简洁的方法。既然我们可以为除指针和接口外的任何类型定义方法，我们可以为函数定义方法。`http` 包包含以下代码：

```go
// HandlerFunc 类型是一个适配器，允许普通函数作为 HTTP 处理程序
// 如果 f 是具有适当签名的函数，HandlerFunc(f) 就是一个调用 f 的 Handler 对象
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP 调用 f(w, req)
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

`HandlerFunc` 是一个带有 `ServeHTTP` 方法的类型，因此该类型的值可以处理 HTTP 请求。看看方法的实现：接收器是一个函数 `f`，而方法调用 `f`，这看起来可能有些奇怪，但与接收器是通道而方法在通道上发送数据并没有太大不同

要将 `ArgServer` 变成 HTTP 服务器，我们首先修改它以具有正确的签名：

```go
// 参数服务器
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

现在 `ArgServer` 具有与 `HandlerFunc` 相同的签名，因此可以转换为该类型以访问其方法，就像我们将 `Sequence` 转换为 `IntSlice` 以访问 `IntSlice.Sort` 一样。设置代码很简洁：

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

当有人访问 `/args` 页面时，安装在该页面的处理程序值为 `ArgServer`，类型为 `HandlerFunc`。HTTP 服务器将调用该类型的 `ServeHTTP` 方法，以 `ArgServer` 作为接收器，进而调用 `ArgServer`（通过 `HandlerFunc.ServeHTTP` 内部的 `f(w, req)` 调用），参数就会显示出来

在本节中，我们从结构体、整数、通道和函数创建了 HTTP 服务器，这一切都是因为接口只是一组方法，这些方法可以（几乎）为任何类型定义

## The blank identifier

我们已经在 `for range` 循环和映射的上下文中多次提到空白标识符。空白标识符可以赋值为任何类型的任何值，这些值会被无害地丢弃。这有点像写入 Unix 的 `/dev/null` 文件：它代表一个只写值，用作占位符，用于需要变量但实际值无关紧要的地方。它的用途不止我们目前看到的这些

### The blank identifier in multiple assignment

`for range` 循环中使用空白标识符是一般情况下的一个特例：多重赋值

如果赋值需要左侧有多个值，但其中一个值不会被程序使用，在赋值左侧使用空白标识符可以避免创建虚拟变量，并清楚地表明该值将被丢弃。例如，当调用一个返回值和错误的函数，但只有错误重要时，使用空白标识符丢弃无关的值：

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
    fmt.Printf("%s does not exist\n", path)
}
```

偶尔你会看到丢弃错误值以忽略错误的代码；这是一种糟糕的做法。始终检查错误返回；它们被提供是有原因的：

```go
// 不好！如果路径不存在，这段代码会崩溃
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a目录\n", path)
}
```

### Unused imports and variables

导入包或声明变量而不使用是一种错误。未使用的导入会使程序臃肿并减慢编译速度，而初始化但未使用的变量至少是计算浪费，可能表明存在更大的错误。然而，当程序处于积极开发阶段时，经常会出现未使用的导入和变量，为了继续编译而删除它们可能会很烦人，尤其是稍后可能再次需要它们。空白标识符提供了一种解决方案

这个半成品程序有两个未使用的导入（`fmt` 和 `io`）和一个未使用的变量（`fd`），因此它不会编译，但能看到目前的代码是否正确会很好：

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: 使用 fd
}
```

要消除关于未使用导入的抱怨，可以使用空白标识符引用导入包中的符号。同样，将未使用的变量 `fd` 赋值给空白标识符可以消除未使用变量的错误。这个版本的程序可以编译：

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // 用于调试；完成后删除
var _ io.Reader    // 用于调试；完成后删除

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: 使用 fd
    _ = fd
}
```

按照惯例，用于消除导入错误的全局声明应紧跟在导入之后并有注释，既便于找到它们，也提醒以后清理。

### Import for side effect

像前面例子中 `fmt` 或 `io` 这样的未使用导入最终应该被使用或移除：空白赋值将代码标识为进行中的工作。但有时仅为了包的副作用而导入包是有用的，无需显式使用。例如，在它的 `init` 函数中，`net/http/pprof` 包注册了提供调试信息的 HTTP 处理程序。它有一个公开的 API，但大多数客户端只需要处理程序注册并通过网页访问数据。要仅为副作用导入包，将包重命名为空白标识符：

```go
import _ "net/http/pprof"
```

这种导入形式清楚地表明包是为了副作用而导入的，因为没有其他可能的用途：在这个文件中，它没有名称（如果有名称而我们不使用该名称，编译器会拒绝程序）

### Interface checks

正如我们在接口讨论中看到的，类型不需要显式声明它实现了某个接口。相反，类型只需实现接口的方法就自动实现了该接口。在实践中，大多数接口转换是静态的，因此在编译时检查。例如，将 `*os.File` 传递给期望 `io.Reader` 的函数，除非 `*os.File` 实现了 `io.Reader` 接口，否则不会编译

但有些接口检查确实发生在运行时。一个例子是 `encoding/json` 包，它定义了 `Marshaler` 接口。当 JSON 编码器接收到实现该接口的值时，编码器调用值的编组方法将其转换为 JSON，而不是执行标准转换。编码器在运行时使用类型断言检查此属性：

```go
m, ok := val.(json.Marshaler)
```

如果只需要询问类型是否实现接口，而不实际使用接口本身，也许作为错误检查的一部分，可以使用空白标识符忽略类型断言的值：

```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("值 %v 的类型 %T 实现了 json.Marshaler\n", val, val)
}
```

这种情况的一个常见场景是，需要在实现类型的包内保证它确实满足接口。如果一个类型（例如 `json.RawMessage`）需要自定义 JSON 表示，它应该实现 `json.Marshaler`，但没有静态转换会使编译器自动验证这一点。如果该类型无意中未能满足接口，JSON 编码器仍将工作，但不会使用自定义实现。要保证实现正确，可以在包中使用空白标识符的全局声明：

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

在这个声明中，涉及将 `*RawMessage` 转换为 `Marshaler` 的赋值要求 `*RawMessage` 实现 `Marshaler`，这一属性将在编译时检查。如果 `json.Marshaler` 接口发生变化，这个包将不再编译，我们会注意到需要更新

空白标识符在这种构造中的出现表明声明仅用于类型检查，而不是创建变量。但不要为每个满足接口的类型都这样做。按照惯例，这种声明仅用于代码中尚未存在静态转换的情况，这是罕见的事件

## Embedding

Go 不提供典型的类型驱动的子类化概念，但它确实有通过嵌入类型到结构体或接口中来"借用"实现的能力。

**接口嵌入**非常简单。我们之前提到过 `io.Reader` 和 `io.Writer` 接口：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

`io` 包还导出了其他几个接口，指定了可以实现多个方法的对象。例如，`io.ReadWriter` 是一个包含 `Read` 和 `Write` 的接口。我们可以通过显式列出这两个方法来指定 `io.ReadWriter`，但嵌入两个接口来形成新接口更容易且更直观：

```go
// ReadWriter 是组合了 Reader 和 Writer 的接口
type ReadWriter interface {
    Reader
    Writer
}
```

这意味着：`ReadWriter` 可以做 `Reader` 和 `Writer` 能做的事；它是嵌入接口的联合。只有接口可以嵌入到接口中。

**结构体嵌入**的基本思想相同，但影响更深远。`bufio` 包有两个结构体类型：`bufio.Reader` 和 `bufio.Writer`，每个都实现了 `io` 包中的相应接口。`bufio` 还实现了一个缓冲的读/写器，它通过使用嵌入将读取器和写入器组合到一个结构体中：

```go
// ReadWriter 存储指向 Reader 和 Writer 的指针
// 它实现了 io.ReadWriter
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

嵌入的元素是指向结构体的指针，当然在使用前必须初始化为指向有效的结构体。`ReadWriter` 结构体也可以写成：

```go
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

但这样为了提升字段的方法并满足 `io` 接口，我们还需要提供转发方法：

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

通过直接嵌入结构体，我们避免了这种簿记工作。嵌入类型的方法会免费提供，这意味着 `bufio.ReadWriter` 不仅具有 `bufio.Reader` 和 `bufio.Writer` 的方法，还满足所有三个接口：`io.Reader`、`io.Writer` 和 `io.ReadWriter`

**嵌入与子类化的重要区别**：当我们嵌入一个类型时，该类型的方法成为外部类型的方法，但当调用它们时，方法的接收器是内部类型，而不是外部类型。在我们的例子中，当调用 `bufio.ReadWriter` 的 `Read` 方法时，其效果与上面写出的转发方法完全相同；接收器是 `ReadWriter` 的 `reader` 字段，而不是 `ReadWriter` 本身

**嵌入也可以是简单的便利**。这个例子显示了一个嵌入字段与一个常规命名字段并存：

```go
type Job struct {
    Command string
    *log.Logger
}
```

现在 `Job` 类型具有 `*log.Logger` 的 `Print`、`Printf`、`Println` 等方法。当然，我们可以给 `Logger` 一个字段名，但这不是必需的。一旦初始化，我们就可以向 `Job` 记录日志：

```go
job.Println("starting now...")
```

`Logger` 是 `Job` 结构体的常规字段，所以我们可以在 `Job` 的构造函数中以通常的方式初始化它：

```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

或者使用复合字面量：

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

如果我们需要直接引用嵌入字段，字段的类型名（忽略包限定符）作为字段名，就像 `ReadWriter` 结构体的 `Read` 方法中那样。这里，如果我们需要访问 `Job` 变量 `job` 的 `*log.Logger`，我们会写 `job.Logger`，如果我们想细化 `Logger` 的方法，这会很有用：

```go
func (job *Job) Printf(format string, args ...interface{}) {
    job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

**嵌入类型引入了名称冲突问题**，但解决规则很简单：

1. 字段或方法 `X` 隐藏了类型更深嵌套部分中的任何其他项 `X`。如果 `log.Logger` 包含名为 `Command` 的字段或方法，`Job` 的 `Command` 字段将优先

2. 如果相同名称出现在相同的嵌套级别，通常是错误的；如果 `Job` 结构体包含另一个名为 `Logger` 的字段或方法，嵌入 `log.Logger` 将是错误的。但是，如果重复名称在类型定义之外的程序中从未被提及，那么是可以的。这个限定提供了针对从外部嵌入类型所做更改的一些保护；如果一个字段与另一个子类型中的另一个字段冲突，但两个字段都从未使用过，那么就没有问题