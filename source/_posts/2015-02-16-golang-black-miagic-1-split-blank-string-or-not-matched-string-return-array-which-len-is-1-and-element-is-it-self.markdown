---
layout: post
title: "golang 黑魔法之一: 使用strings.Split()函数时，如果string不包含sep则返回[string]而不是[]"
date: 2015-02-16 10:43:52 +0800
comments: true
categories: golang
---

学习golang一小段时间了，深深被其敢于改变的特性所吸引。但有时也不免改动过大，让人难以理解

这是一个随手笔记，可能会有错误。。。。。

直接上代码

    package main

    import (
        "fmt"
        "strings"
    )

    func main() {
        fmt.Println(strings.Split("", "ss"))
        fmt.Println(strings.Split("a", "ss"))
    }

它将有如下输出

    [""]
    ["a"]

所以在golang中认为split失败时需要返回一个Array，这个Array的元素是其本身。其他语言(至少在我的认识中)则认为返回一个空Array。这是对我而言一个坑。因为如果这样使用，会有问题

    package main

    import (
        "fmt"
        "strings"
    )

    func main() {
        fmt.Println(strings.Split("", "ss"))
        for _, word := range strings.Split("a", "ss") {
            fmt.Println(word)
        }
    }

我希望它是输入(所有的双引号只是用来区分字符串长度什么的，真正的输出是不会有双引号的)

    0
    "" //空字符串

但它的输出会是这个样子

    1
    "a"

所以这个坑暂时没有解决办法，有想法的可以交流下~~~

[Golang关于这个东西的讨论](https://groups.google.com/forum/#!topic/golang-nuts/S7U1iRmuydg)
