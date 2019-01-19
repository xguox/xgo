---
title: "Go 的指针"
date: 2018-11-14T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

与 Ruby 一样, Go 在调用函数(方法)的时候都是值传递的, 即参数的副本. 在 Go 里边, 即使传的是指针类型的变量也一样, 实际上传的是指针的副本, 指向的是同样的内存地址. 因此调用时候, 传指针可能就会改变参数变量的值.

此外, 当把元素塞进 `array`, `slice`, `map`, `channel` 的时候也是类似的道理. [查看示例](https://xguox.me/golang-pointer-array-slice-map-copy.html)

> **[As in all languages in the C family, everything in Go is passed by value.](https://golang.org/doc/faq#different_method_sets) That is, a function always gets a copy of the thing being passed, as if there were an assignment statement assigning the value to the parameter. For instance, passing an int value to a function makes a copy of the int, and passing a pointer value makes a copy of the pointer, but not the data it points to.**

还有, 作为方法的接收者和返回值也都是. 定义方法时候用值还是指针可以参考以下几个因素:

- **First, and most important**, 该方法是否会修改接收者的值.
- **Second is the consideration of efficiency.**, 如果接收者是一个「巨大」的结构体对象, 那还是用指针吧.
- **Next is consistency.** 如果同一个结构体下有其他方法是指针接收者的话, 那就统一全都用指针吧.
- 其余, 如基础类型, slices, 或小型结构体则用值作为接收者更高效, 清晰

***T** 的方法集(method set)包含所有接收者是 *T 以及接收者为 T 的方法. 但是, 反过来是不成立的, **T** 所拥有的方法集仅限于接收者为 T.

之所以会出现这种区别, 是因为如果一个接口值是 *T, 调用方法时候 Go 可以自动取指针值完成调用, 但是, 如果接口值是 T, 是没有安全的方式取得 T 值的指针[因为这样做的话方法就可以修改接口(不是类型)的值, 这是 Go 的语言规范所不允许的]

> Even in cases where the compiler could take the address of a value to pass to the method, if the method modifies the value the changes will be lost in the caller. As an example, if the Write method of bytes.Buffer used a value receiver rather than a pointer, this code:

```go
var buf bytes.Buffer
io.Copy(buf, os.Stdin)
```

would copy standard input into a copy of buf, not into buf itself. This is almost never the desired behavior.

