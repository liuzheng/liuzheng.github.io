---
layout: post
title: "突破限制,访问其它Go package中的私有函数"
tagline: ""
description: ""
category: 
tags: ["go"]
---

转自：<https://colobu.com/2017/05/12/call-private-functions-in-other-packages/>

熟悉C++、Java、C#等面向对象的编程语言的同学，在学习Go语言的过程中，经常会被访问权限所困扰，逐渐才能了解这样一个事实：

*Go语言通过`identifier`的首字母是否大写来决定它是否可以被其它package所访问。*

正式的Go语言规范是这么规定的：

```
An identifier may be exported to permit access to it from another package. An identifier is exported if both:

the first character of the identifier's name is a Unicode upper case letter (Unicode class "Lu"); and
the identifier is declared in the package block or it is a field name or method name.

All other identifiers are not exported.

```

这个Go语言规范定义的访问权限控制方法。

但是有没有办法突破这个限制呢？

突破可以从两个方向来讨论： 将`exported`类型变为其它package不可访问；将`unexported`的类型变为其它package可访问。

# 将`exported`类型变为其它package不可访问


至少有一个办法可以将package中 exported的函数、类型变为其它package不可访问， 那就是定义一个`internal` package,将这些package放在`internal` package之下。

Go语言本身没有这个限制，这是通过`go`命令实现的。最早这个特性是在 *go 1.4*版本中引入的，相关的细节可以查看文档： [design document](https://docs.google.com/document/d/1e8kOo3r51b2BWtTs_1uADIA5djfXhPT36s6eHVRIvaU/edit)

这个规则是这样的：

```
An import of a path containing the element “internal” is disallowed if the importing code is outside the tree rooted at the parent of the “internal” directory.
```

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






















































