---
title: "Go 的修饰模式(Decorator Pattern)"
date: 2018-09-28T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

Decorator(修饰, 装饰, 虽然都是这么翻译, 但是怎么看怎么别扭, 所以还是用英文好了) Pattern, 是之前提到的代理模式(Proxy Pattern)的大兄弟. 用起来挺相似的. 都是在不改变原本的旧有类型的代码前提下, 对该类型的功能进行拓展或者修改.

[https://zh.wikipedia.org/wiki/修饰模式#介绍](https://zh.wikipedia.org/wiki/%E4%BF%AE%E9%A5%B0%E6%A8%A1%E5%BC%8F#%E4%BB%8B%E7%BB%8D)
> 通过使用修饰模式，可以在运行时扩充一个类的功能。原理是：增加一个修饰类包裹原来的类，包裹的方式一般是通过在将原来的对象作为修饰类的构造函数的参数。装饰类实现新的功能，但是，在不需要用到新功能的地方，它可以直接调用原来的类中的方法。修饰类必须和原来的类有相同的接口。

> 修饰模式是类继承的另外一种选择。类继承在编译时候增加行为，而装饰模式是在运行时增加行为。

> 当有几个相互独立的功能需要扩充时，这个区别就变得很重要。在有些面向对象的编程语言中，类不能在运行时被创建，通常在设计的时候也不能预测到有哪几种功能组合。这就意味着要为每一种组合创建一个新类。相反，修饰模式是面向运行时候的对象实例的,这样就可以在运行时根据需要进行组合。

```go
package main

import "fmt"

// step1: 编写基础功能，刚开始不需要定义接口
type Base struct {
}
func (b *Base) Call() string {
    return "base is called"
}

// step2: 将上面的方法声明为接口类型，基础功能中的 Call() 调用自动满足下面的接口
type DecoratorI interface {
    Call() string
}

// step3: 编写新增功能，结构中保存接口类型的参数
type Decorator struct {
    derorator DecoratorI
}

func (d *Decorator) Call() string {
    return "decorator: " + d.derorator.Call()
}

func main() {
    base := &Base{}
    fmt.Println(base.Call())

    decorator := Decorator{base}
    fmt.Println(decorator.Call())
}
```

咋一看其实还真跟 Proxy Pattern 就是一个东西的样子, 但是, 其实 Decorator Pattern 胜在更灵活一些. Proxy Pattern 在编译时候就要定义好新功能的样子并且不能改变, 而 Decorator Pattern 可以在运行时动态的根据需要进行组合. 换一个纬度说, Proxy Pattern 更像是对旧有类型做的一个访问控制.

-------------

##### Decorator Pattern 的实际用例

```go
type MyServer struct{}

func (m *MyServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello Decorator!")
}


func main() {
    http.Handle("/", &MyServer{})
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

加上 Logger 中间件, 修饰这个 `MyServer`

```go
type LoggerServer struct {
    Handler   http.Handler
    LogWriter io.Writer
}
func (s *LoggerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(s.LogWriter, "Request URI: %s\n", r.RequestURI)
    fmt.Fprintf(s.LogWriter, "Host: %s\n", r.Host)
    fmt.Fprintf(s.LogWriter, "Content Length: %d\n", r.ContentLength)
    fmt.Fprintf(s.LogWriter, "Method: %s\n", r.Method)fmt.Fprintf(s.LogWriter, "--------------------------------\n")
    s.Handler.ServeHTTP(w, r)
}

func main() {
    http.Handle("/", &LoggerServer{
        LogWriter:os.Stdout,
        Handler:&MyServer{},
    })
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

如果需要可以再继续嵌套更多的中间件, 最后都是调用 `s.Handler.ServeHTTP(w, r)`