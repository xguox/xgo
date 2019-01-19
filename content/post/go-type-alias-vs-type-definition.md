---
title: "Golang 类型声明与类型别名"
date: 2018-10-09T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

Go 可以给任何的常量, 变量, 函数, 类型设置别名. ~~对于写惯 Ruby 的 Alias(其实用的也不多), 第一感觉应该是不带等号的才是类型别名, 然后, Go 的累习惯别名刚好相反.~~

```go
import "fmt"

type A struct{}
type AliasA = A // 类型别名
type B A        // 类型定义

type C int
type D = int

func (a AliasA) testA() {
    fmt.Println("test A")
}

func (b B) testB() {
    fmt.Println("test B")
}

func (c *C) testC() {
    fmt.Println("test C")
}

// cannot define new methods on non-local type int

// func (d *D) testD() {
// 	fmt.Println("test D")
// }

func main() {
    var a = A{}
    var aa AliasA = a
    // var b1 B = a // cannot use a (type A) as type B in assignment
    var b2 B
    a.testA()  // test A
    aa.testA() // test A
    b2.testB() // test B

    var c C
    c.testC()

    // var d D
    // d.testD() // d.testD undefined (type int has no field or method testD)

}

```

**如果是被别名的类型(= . = )跟别名不是再同一个包的话, 则不能为其添加新的方法**, 比如这里 `type D = int`, 不能再给 D 定义任何的方法了, 因为 D 和基础数据类型 `int` 不在同一个包里边. (我理解的**non-local type**)

**类型声明与类型别名最大区别在于: 类型别名和原类型是相同的, 而类型声明和原类型则是完全不同的两个东西**, 只不过, 类型声明的新类型拥有与原类型相同的字段结构, 但, 不拥有任何原类型的方法.

- [https://go.googlesource.com/proposal/+/master/design/16339-alias-decls.md](https://go.googlesource.com/proposal/+/master/design/16339-alias-decls.md)