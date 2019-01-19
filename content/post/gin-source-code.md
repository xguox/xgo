---
title: "从 net/http 入门到 Gin 源码梳理"
date: 2019-01-10T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

通过 Go 的标准库 `net/http` 可以轻松几行运行起一个简单的 web 服务, 比如:

```go
package main

import "net/http"

func main() {
    http.Handle("/", &MyHandler{})
    http.Handle("/hello", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("world"))
    }))
    http.HandleFunc("/ping", hello)

    http.ListenAndServe(":8080", nil)
}

func hello(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("pong"))
}

type MyHandler struct{}

func (m *MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("MyHandler ServeHTTP"))
}
```

上面这段代码分别用了不太一样的方式注册了三个简单的路由路径及相应的请求处理. 说不太一样其实还是有共通的地方的.
其实就是两个 `net/http` 包里边的函数

```go
func Handle(pattern string, handler Handler)

func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```

这两个函数都接受两个参数, 第一个都是请求的路径 pattern, **Handle** 的第二个参数是实现了 **Handler** 接口的类型, **HandleFunc** 的第二个参数则是一个**函数类型的参数**, 只要传入的函数签名是 `func(http.ResponseWriter, *http.Request)` 就可以了. 所以, 其实第三个路由方式还可以写成下面这样:

```go
m := MyHandler{}
http.HandleFunc("/ping", m.ServeHTTP)
// ServeHTTP 的签名能匹配得上
```

即使第二个参数不一样, 但实际要做的事情还是一样的, 最后都是丢给 **DefaultServeMux** 完成请求的处理.

```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    if handler == nil {
    panic("http: nil handler")
    }
    mux.Handle(pattern, HandlerFunc(handler))
}
```

比较特别一些的是第二个路由,

```go
http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("world"))
})
```

这里把一个匿名函数转换成为了 **http.HandlerFunc** 类型, 而 **http.HandlerFunc** 类型实现了 **http.Handler**, 因此, 也是一个正确的使用方式.

**http.HandlerFunc** 是一个等价于 `func(ResponseWriter, *Request)` 的类型. ([每次看到这类写法都要细细想一下是类型定义还是类型别名, 以及有啥不一样, 然后...又慢慢的忘了](https://xguox.me/go-type-alias-vs-type-definition.html))

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers. If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

----------

#### Gin

[Gin](https://github.com/gin-gonic/gin) 是基于 `net/http` 的 web framework(确切一丢说是基于 [httprouter](https://github.com/julienschmidt/httprouter)). 相较于 `net/http` 默认的路由功能, **httprouter** 提供了更好的拓展性([基于 trie 的路由结构](https://xguox.me/gin-router-conflicts.html)). **httprouter** 的 github 主页上强调了几次 **scale**, P.S. 这里[有一篇站队 `net/http` 的路由功能的文章](https://blog.merovius.de/2017/06/18/how-not-to-use-an-http-router.html)).

类似上面的三个请求服务, 下面是用 Gin 所写的版本(没有使用任何 middleware)

```go
package main

import (
    "github.com/gin-gonic/gin"
    "net/http"
)

func main() {
    r := gin.New()
    r.GET("/", func(context *gin.Context) {
    context.String(http.StatusOK, "gin home page")
    })
    r.GET("/ping", func(context *gin.Context) {
    context.String(http.StatusOK, "pong")
    })
    r.GET("/hello", func(context *gin.Context) {
    context.String(http.StatusOK, "world")
    })
    r.Run()
}
```

丢完示例以后, 跟着示例代码大概的走一遍 Gin 的全流程. 之后在深入看 RouterGroup 和 middleware 相关的探究.

-------

示例代码用的是 `gin.New()` 初始化得到一个 `*gin.Engine`, 这个 Engine 是不带任何 middleware 的, 有时候还会使用 `gin.Default()` 来初始化, 其实就是空的 Engine 加上了 **Logger** 和 **Recovery** 这俩 middleware 了.

```go
func Default() *Engine {
    debugPrintWARNINGDefault()
    engine := New()
    engine.Use(Logger(), Recovery())
    return engine
}

```

**gin.Engine** 的结构体定义:

```go
type Engine struct {
    RouterGroup
    // ...好吧, 省略一大票
    trees   methodTrees // 路由 radix trie 数组
}
```

RouterGroup 是其中的一个嵌套类型, 所以, 前面的示例中:

```go
r.GET("/", func(context *gin.Context) {
    context.String(http.StatusOK, "gin home page")
})
```

类似的 `r.GET`, `r.POST`, `r.DELETE` 其实调用的是 **gin.Engine** 中的 `RouterGroup` 对应的方法.

```go
// GET is a shortcut for router.Handle("GET", path, handle).
func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes {
    return group.handle("GET", relativePath, handlers)
}

func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
    absolutePath := group.calculateAbsolutePath(relativePath)
    handlers = group.combineHandlers(handlers)
    group.engine.addRoute(httpMethod, absolutePath, handlers)
    return group.returnObj()
}
```

大概就是注册路由相关的事情, 整理好路径, 合并 handlers, 最后加入到相应的路由表(radix tree).

然后就是, `r.Run()`,

```go
func (engine *Engine) Run(addr ...string) (err error) {
    defer func() { debugPrintError(err) }()

    address := resolveAddress(addr)
    debugPrint("Listening and serving HTTP on %s\n", address)
    err = http.ListenAndServe(address, engine)
    return
}
```

其实就还是 `net/http` 的那套 `func ListenAndServe(addr string, handler Handler)`, 只不过传入的是 **gin.Engine**, 替代了 **http.DefaultServeMux** (当ListenAndServe 第二个参数为 nil 的时候).

```go
// ServeHTTP conforms to the http.Handler interface.
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    c := engine.pool.Get().(*Context)
    c.writermem.reset(w)
    c.Request = req
    c.reset()

    engine.handleHTTPRequest(c)

    engine.pool.Put(c)
}
```

之后就是 `net/http` 包的 `server.go` 那一套了. TCP Scoket 的监听, 请求与响应.

```go
// net/http/server.go

func ListenAndServe(addr string, handler Handler) error {
    server := &Server{Addr: addr, Handler: handler}
    return server.ListenAndServe()
}

func (srv *Server) ListenAndServe() error {
    // ...
    ln, err := net.Listen("tcp", addr) // 监听 TCP(HTTP) 的连接
    if err != nil {
    return err
    }
    return srv.Serve(tcpKeepAliveListener{ln.(*net.TCPListener)})
}

func (srv *Server) Serve(l net.Listener) error {
    // ...
    defer l.Close()

    // ...
    for {
    rw, e := l.Accept() // 阻塞等待客户端连接
    // ...
    c := srv.newConn(rw)
    c.setState(c.rwc, StateNew) // before Serve can return
    go c.serve(ctx) // 起一个 goroutine 处理接受的请求
    }
}

func (c *conn) serve(ctx context.Context) {
    // ...
    for {
    w, err := c.readRequest(ctx)
    // ...
    serverHandler{c.server}.ServeHTTP(w, w.req)
    // ...
    }
}

// serverHandler delegates to either the server's Handler or
// DefaultServeMux and also handles "OPTIONS *" requests.
type serverHandler struct {
    srv *Server
}

func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
    handler := sh.srv.Handler  // gin.Engine
    if handler == nil {
    handler = DefaultServeMux
    }
    if req.RequestURI == "*" && req.Method == "OPTIONS" {
    handler = globalOptionsHandler{}
    }
    handler.ServeHTTP(rw, req)
}
```

最后这个 handler 就是一路带进来的 **gin.Engine**, 回到前面贴过的 `func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request)`

```go
// ServeHTTP conforms to the http.Handler interface.
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    c := engine.pool.Get().(*Context)
    c.writermem.reset(w)
    c.Request = req
    c.reset()

    engine.handleHTTPRequest(c) // 通过请求 method 找到 engine.trees 中对应的树 -> 在树中查找对应的路由, 执行相关的 handlers
    engine.pool.Put(c)
}
```

接下来就是遍布 Gin 项目的 **gin.Context** 登场. 源码注释也说了, Context 是 Gin 最重要的一部分. 它可以让我们在 middleware 之间传递变量, 管理整个请求/响应流程, 比如验证请求的 JSON 以及返回一个 JSON 响应等等.

```go
type Context struct {
    writermem responseWriter
    Request   *http.Request
    Writer    ResponseWriter

    Params   Params
    handlers HandlersChain
    index    int8

    engine *Engine

    Keys map[string]interface{}

    Errors errorMsgs

    Accepted []string
}

// ...

func (c *Context) Next() {
    c.index++
    for s := int8(len(c.handlers)); c.index < s; c.index++ {
    c.handlers[c.index](c) // 执行 handler
    }
}

```

可以看出其实主要是对 `func(w http.ResponseWriter, r *http.Request)` 的一个封装, 其中用于响应的 `ResponseWriter` 基于 `http.ResponseWriter` 做了些拓展.

```go
type responseWriterBase interface {
    http.ResponseWriter
    http.Hijacker
    http.Flusher
    http.CloseNotifier

    // ...
}

type responseWriter struct {
    http.ResponseWriter
    size   int
    status int
}
```

**gin.Context** 中的一大票返回 Format 方法

![](http://wx4.sinaimg.cn/large/62fdd4d5gy1fz0q52gh3bj212w14qguf.jpg)


```go
func (c *Context) String(code int, format string, values ...interface{}) {
    c.Render(code, render.String{Format: format, Data: values})
}
```

------------

Related:

- [Gin RouterGroup 与 middleware 相关源码](https://xguox.me/gin-RouterGroup-and-middleware-walkthrough.html)
- [https://chai2010.gitbooks.io/advanced-go-programming-book/content/](https://chai2010.gitbooks.io/advanced-go-programming-book/content/)
- [https://stackoverflow.com/questions/49668070/how-does-servehttp-work](https://stackoverflow.com/a/49674486/1251496)