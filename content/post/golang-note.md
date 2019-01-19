---
title: "Golang 笔记"
date: 2018-07-31T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

- main 函数保存在名为 main 的包里. 如果 main 函数不在 main 包里, 构建工具就不会生成可执行的文件.

- 常量只能是 **numeric** 或者 **string**

> There are boolean constants, rune constants, integer constants, floating-point constants, complex constants, and string constants. Rune, integer, floating-point, and complex constants are collectively called numeric constants.

- `:=` 只能在函数/方法中使用

> non-declaration statement outside of function body

- `[arrLength]int` 初始化 array 的容量不能通过计算得出, 需要计算得出的话得用 make, `left := make([]int, arrLength)` 否则会报 **non-constant array bound**

- Go语言中的源代码定义为 UTF-8 文本, 不允许其他的表示. 在代码中写下字符 ``` `⌘` ``` 时, 用于创建程序的文本编辑器将符号 ⌘ 的 UTF-8 编码放入文本中. 当打印十六进制字节时, 我们只是将文件中的数据打印出来.

- `runtime.GOMAXPROCS(1)` 不代表就是单线程. 如果 Golang 运行时只分配一个逻辑处理器给调度器使用(`runtime.GOMAXPROCS(1)`)来执行所有 goroutine. 即使只有这一个逻辑处理器也可以让多个 goroutine 并发运行.

> 基于调度器的内部算法, 一个正运行的goroutine在工作结束前, 可以被停止并重新调度. 调度器这样做的目的是防止某个goroutine长时间占用逻辑处理器. 当goroutine占用时间过长时, 调度器会停止当前正运行的goroutine, 并给其他可运行的goroutine运行的机会.

- Conversions

> Conversions are expressions of the form T(x) where T is a type and x is an expression that can be converted to type T.

> If the type starts with the operator * or <-, or if the type starts with the keyword func and has no result list, it must be parenthesized when necessary to avoid ambiguity:

```go
*Point(p)        // same as *(Point(p))
(*Point)(p)      // p is converted to *Point
<-chan int(c)    // same as <-(chan int(c))
(<-chan int)(c)  // c is converted to <-chan int
func()(x)        // function signature func() x
(func())(x)      // x is converted to func()
(func() int)(x)  // x is converted to func() int
func() int(x)    // x is converted to func() int (unambiguous)
```

Converting a constant yields a typed constant as result.

```go
uint(iota)               // iota value of type uint
float32(2.718281828)     // 2.718281828 of type float32
complex128(1)            // 1.0 + 0.0i of type complex128
float32(0.49999999)      // 0.5 of type float32
float64(-1e-1000)        // 0.0 of type float64
string('x')              // "x" of type string
string(0x266c)           // "♬" of type string
MyString("foo" + "bar")  // "foobar" of type MyString
string([]byte{'a'})      // not a constant: []byte{'a'} is not a constant
(*int)(nil)              // not a constant: nil is not a constant, *int is not a boolean, numeric, or string type
int(1.2)                 // illegal: 1.2 cannot be represented as an int
string(65.0)             // illegal: 65.0 is not an integer constant
```

A non-constant value x can be converted to type T in any of these cases:

- x is assignable to T.
- ignoring struct tags (see below), x's type and T have identical underlying types.
- ignoring struct tags (see below), x's type and T are pointer types that are not defined types, and their pointer base types have identical underlying types.
- x's type and T are both integer or floating point types.
- x's type and T are both complex types.
- x is an integer or a slice of bytes or runes and T is a string type.
- x is a string and T is a slice of bytes or runes.