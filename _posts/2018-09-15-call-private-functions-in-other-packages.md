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

Go语言本身没有这个限制，这是通过`go`命令实现的。最早这个特性是在 *go 1.4*版本中引入的，相关的细节可以查看文档： (design document)[https://docs.google.com/document/d/1e8kOo3r51b2BWtTs_1uADIA5djfXhPT36s6eHVRIvaU/edit]

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

如果你查看 Go 标准库的的代码， 比如 (time/sleep.go)[https://github.com/golang/go/blob/master/src/time/sleep.go] 文件， 你会发现一些奇怪的函数， 如 `Sleep`:

```
func Sleep(d Duration)
```



























































