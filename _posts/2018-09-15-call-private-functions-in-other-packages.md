---
title: "突破限制,访问其它Go package中的私有函数"
tagline: ""
description: ""
categorys: 
tags: ["go"]
---

转自：<https://colobu.com/2017/05/12/call-private-functions-in-other-packages/>

熟悉C++、Java、C#等面向对象的编程语言的同学，在学习Go语言的过程中，经常会被访问权限所困扰，逐渐才能了解这样一个事实：

**Go语言通过`identifier`的首字母是否大写来决定它是否可以被其它package所访问。**

正式的Go语言规范是这么规定的：

 > An identifier may be exported to permit access to it from another package. An identifier is exported if both:
 > 
 > the first character of the identifier's name is a Unicode upper case letter (Unicode class "Lu"); and the identifier is declared in the package block or it is a field name or method name.
 > 
 > All other identifiers are not exported.

这个Go语言规范定义的访问权限控制方法。

但是有没有办法突破这个限制呢？

突破可以从两个方向来讨论： 将`exported`类型变为其它package不可访问；将`unexported`的类型变为其它package可访问。

# 将`exported`类型变为其它package不可访问


至少有一个办法可以将package中 exported的函数、类型变为其它package不可访问， 那就是定义一个`internal` package,将这些package放在`internal` package之下。

Go语言本身没有这个限制，这是通过`go`命令实现的。最早这个特性是在 **go 1.4**版本中引入的，相关的细节可以查看文档： [design document](https://docs.google.com/document/d/1e8kOo3r51b2BWtTs_1uADIA5djfXhPT36s6eHVRIvaU/edit)

这个规则是这样的：

 > An import of a path containing the element “internal” is disallowed if the importing code is outside the tree rooted at the parent of the “internal” directory.

也就是`internal`包下的 exported 类型只能由internal所在的package (internal的parent)为root的package所访问。

举例来说：

 - `/a/b/c/internal/d/e/f` 可以被`/a/b/c` import， 不能被 `/a/b/g` import.
 - `$GOROOT/src/pkg/internal/xxx`只可以被标准库`$GOROOT/src/*` import.
 - `$GOROOT/src/pkg/net/http/internal` 只可以被 `net/http` 和 `net/http/*` import.
 - `$GOPATH/src/mypkg/internal/foo` 只能被`$GOPATH/src/mypkg import`.

# 访问其它package中的私有方法

如果你查看 Go 标准库的的代码， 比如 [time/sleep.go](https://github.com/golang/go/blob/master/src/time/sleep.go) 文件， 你会发现一些奇怪的函数， 如 `Sleep`:

```go
func Sleep(d Duration)
```

这个函数我们经常会用到， 也就是`time.Sleep`函数，但是这个函数并没有函数体，而且同样的目录下也没有汇编语言的代码实现，那么，这个函数在哪里定义的？

依照[规范](https://golang.org/ref/spec#Function_declarations)，一个只有函数声明的函数是在Go的外部实现的，我们称之为`external function`。

实际上，这个"外部函数"也是在Go标准库中实现的，它是 runtime中的一个 unexported的函数:

```go
//go:linkname timeSleep time.Sleep
func timeSleep(ns int64) {
	if ns <= 0 {
		return
	}
	t := getg().timer
	if t == nil {
		t = new(timer)
		getg().timer = t
	}
    ......
}
```

事实上，runtime为其它 package中定义了很多的函数，比如sync、net中的一些函数，你可以通过命令grep linkname /usr/local/go/src/runtime/*.go查找这些函数。

我们会有两个疑问：一是为什么这些函数要定义在 runtime package中，而是这个机制到底是怎么实现的？

将相关的函数定义在runtime中的好处是， 它们可以访问 runtime package中 unexported的类型， 比如getp函数等，相当于往 runtime package打入一个"叛徒",通过"叛徒"可以访问 runtime package 的私有对象。同时，这些"叛徒"函数尽管被声明为unexported,还是可以在其它package中访问。

第二个问题，其实是Go的go:linkname这个指令发挥的作用,它的格式如下：

```go
//go:linkname localname importpath.name
```
[Go文档](https://golang.org/cmd/compile/)说明了这个指令的作用：

 > The //go:linkname directive instructs the compiler to use “importpath.name” as the object file symbol name for the variable or function declared as “localname” in the source code. Because this directive can subvert the type system and package modularity, it is only enabled in files that have imported "unsafe".

这个指令告诉编译器为函数或者变量`localname`使用`importpath.name`作为目标文件的符号名。因为这个指令破坏了类型系统和包的模块化，所以它只能在 import "unsafe" 的情况下才能使用。

`importpath.name`可以是这种格式:`a/b/c/d/apkg.foo`，这样在package `a/b/c/d/apkg`中就可以使用这个函数`foo`了。

举个例子,假设我们的package布局如下:

```
├── a
│   └── a.go
├── b
│   ├── b.go
│   └── internal.s
└── main
    └── main.go
```

package **a** 定义了私有的方法,并加上 `go:linkname`指令， package **b** 可以调用 package **a**的私有方法。 **main.go** 测试访问 **b**中的函数。

首先看看`a.go`中的实现：

```go
package a
import (
	_ "unsafe"
)
//go:linkname say a.say
//go:nosplit
func say(name string) string {
	return "hello, " + name
}
//go:linkname say2 github.com/smallnest/private/b.Hi
//go:nosplit
func say2(name string) string {
	return "hi, " + name
}
```

它定义了两个方法，符号名分别为`a.say`和`github.com/smallnest/private/b.Hi`。

这个不同的符号名的方式会影响**b**中的使用。

```go
package b
import (
	_ "unsafe"
	_ "github.com/smallnest/private/a"
)
//go:linkname say a.say
func say(name string) string
func Greet(name string) string {
	return say(name)
}
func Hi(name string) string
```

在**b** 中，如果想使用符号`a.say`，你还是需要`go:linkname`,告诉编译器这个函数的符号为`a.say`。对于Hi函数， 我们不需要`go:linkname`指令，因为在a.go中我们定义的符号名称**恰巧**就是这个`package.funcname`。

注意，你需要引入package `unsafe`,并且在**b.go**还需要import package a.

你可以在`main.go`中调用**b**:

```go
package main
import (
	"fmt"
	"github.com/smallnest/private/b"
)
func main() {
	s := b.Greet("world")
	fmt.Println(s)
	s = b.Hi("world")
	fmt.Println(s)
}
```

但是，如果你`go run main.go`,你不会得到正确的结果，而是会出错：

```go
main go run main.go
# github.com/smallnest/private/b
../b/b.go:10: missing function body for "say"
../b/b.go:16: missing function body for "Hi"
```

难道我们前面讲的都是错的吗？

这里有一个技巧，你在 package b下创建一个空的文件， 文件名随意，只要文件后缀为`.s`，再运行一下`go run main.go`：

```go
main go run main.go
hello, world
hi, world
```

原因在于Go在编译的时候会启用-complete编译器flag,它要求所有的函数必需包含函数体。创建一个空的汇编语言文件绕过这个限制。

当然， 一般情况下我们不会用到本文所列出的两种突破方式，只有在很稀少的情况下，为了更好地组织我们的代码，我们才会有选择的采用这两种方法。至少，作为一个Go开发者，你会记住有两种突破方法，可以打破Go语言规范中关于权限的限制。

 > 一个应用的例子是可以在代码中访问`sync.runtime_registerPoolCleanup`,因为它有明确的linkname

```go
//go:linkname runtime_registerPoolCleanup sync.runtime_registerPoolCleanup
func runtime_registerPoolCleanup(cleanup func())
```

# 访问其它package中的struct 私有字段
再额外附送一个技巧， 可以访问其它package struct的私有字段。

当然正常情况下struct的私有字段并没有export，所以在其它package是不能正常访问。通过使用refect,可以访问struct的私有字段:

```go
import (
	"fmt"
	"reflect"
	"github.com/smallnest/private/c"
)
func ChangeFoo(f *c.Foo) {
	v := reflect.ValueOf(f)
	x := v.Elem().FieldByName("x")
	fmt.Println(x.Int())
	//panic: reflect: reflect.Value.SetInt using value obtained using unexported field
	//x.SetInt(100)
	fmt.Println(x.Int())
	y := v.Elem().FieldByName("Y")
	y.SetString("world")
	fmt.Println(f.Y)
}
```

但是你不能设置私有字段的值，否则会panic,这是因为`SetXXX`会首先使用`v.mustBeAssignable()`检查字段是否是exported的。

当然，还可以通过"指针"的方式获取字段的地址，通过地址获取数据或者设置数据。
还是用相同的例子：

```go
package c
type Foo struct {
	x int
	Y string
}
func (f Foo) X() int {
	return f.x
}
func New(x int, y string) *Foo {
	return &Foo{x: x, Y: y}
}
```

在package d中访问：

```go
package d
import (
	"fmt"
	"unsafe"
	"github.com/smallnest/private/c"
)
func ChangeFoo(f *c.Foo) {
	p := unsafe.Pointer(f)
	// 事先获取或者通过 reflect获得
	// 本例中是第一个字段，所以offset=0
	offset := uintptr(0)
	ptr2x := (*int)(unsafe.Pointer(uintptr(p) + offset))
	fmt.Println(*ptr2x)
	*ptr2x = 100
	fmt.Println(f.X())
}
```

# 更hack的方法
如果你还不满足，那么我再赠送一个更hack的方法，但是这个也有点限制，就是你要调用的方法应该在之前的某处调用过。

这是 Alan Pierce 提供了一个方法。[runtime/symtab.go](https://golang.org/src/runtime/symtab.go)中保存了符号表，通过一些技巧(`go:linkname`),能访问它的私有方法，查找到想要调用的函数，然后就可以调用了，Alan将相关的代码写成了一个库，方便调用：[go-forceexport](https://github.com/alangpierce/go-forceexport)。

使用方法如下:

```go
var timeNow func() (int64, int32)
err := forceexport.GetFunc(&timeNow, "time.now")
if err != nil {
    // Handle errors if you care about name possibly being invalid.
}
// Calls the actual time.now function.
sec, nsec := timeNow()
```

我在使用的过程中发现只有相应的方法在某处调用过, 符号表中才有这个函数的信息， forceexport.GetFunc才会返回对应的函数。

另外，这是一个非常hack的方式，不保证Go将来的版本是否还能使用，仅供嬉戏之用，慎用在产品代码中。

# 参考文档

 - <https://golang.org/cmd/compile/>
 - <https://github.com/golang/go/issues/15006>
 - <https://siadat.github.io/post/golinkname>
 - <https://sitano.github.io/2016/04/28/golang-private/>
 - <https://golang.org/doc/go1.4#internalpackages>
 - <https://www.alangpierce.com/blog/2016/03/17/adventures-in-go-accessing-unexported-functions/>
 - <https://groups.google.com/forum/#!topic/golang-nuts/ppGGazd9KXI>


# 注

修改转文几处错误表述。

