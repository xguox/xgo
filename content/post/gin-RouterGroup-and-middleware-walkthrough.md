---
title: "Gin RouterGroup 与 middleware 相关源码"
date: 2019-01-11T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

拿[上一篇](https://xguox.me/gin-source-code.html)用的例子改一下, 看看 Gin 的**RouterGroup** 与 **middleware** 是怎么工作的.

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

    admin := r.Group("/admin")
    admin.Use(gin.Logger())
    admin.GET("/hello", func(context *gin.Context) {
        context.String(http.StatusOK, "admin")
    })

    r.Run()
}
```

**New** 函数返回的是一个没有附加任何 middleware 的 **gin.Engine** 指针. 其中这个  Engine 的**嵌套类型 RouterGroup** 被赋值为:

```go
RouterGroup{
    Handlers: nil,
    basePath: "/",
    root:     true,
}
```

**Handlers** 是一组 `gin.HandlerFunc`,

```go
// HandlerFunc defines the handler used by gin middleware as return value.
type HandlerFunc func(*Context)

// HandlersChain defines a HandlerFunc array.
type HandlersChain []HandlerFunc
```

`basePath` 就是这一组路由的路径前缀, 类似 **Rails** 路由中的 scope 或者基于 scope 的 namespace 加上的路径前缀.

跟 `r.GET`, `r.POST` 一样, 其实 `r.Group` 也是 `gin.Engine` 里面的 RouterGroup 嵌套类型的方法. 作用就是构建一个有共通前缀路径的 **RouterGroup**, 并且这组路由**在该前缀路径下**使用着特定的一些 middleware.

```go
func (group *RouterGroup) Group(relativePath string, handlers ...HandlerFunc) *RouterGroup {
    return &RouterGroup{
        Handlers: group.combineHandlers(handlers),
        basePath: group.calculateAbsolutePath(relativePath),
        engine:   group.engine,
    }
}
```

middleware 可以在使用 `Group` 方法时候作为其第二个参数传入进来, 也可以单独使用 `Use` 方法.

```go
    admin := r.Group("/admin", gin.Logger())
    admin.Use(gin.Logger())
```

`Use` 是直接 `append` 到 RouterGroup 的 `Handlers` 最后的, 而 `Group` 会稍复杂一些调用 `group.combineHandlers(handlers)`,

```go
func (group *RouterGroup) combineHandlers(handlers HandlersChain) HandlersChain {
    finalSize := len(group.Handlers) + len(handlers)
    if finalSize >= int(abortIndex) {
        panic("too many handlers")
    }
    mergedHandlers := make(HandlersChain, finalSize)
    copy(mergedHandlers, group.Handlers)
    copy(mergedHandlers[len(group.Handlers):], handlers)
    return mergedHandlers
}
// ...
func (group *RouterGroup) Use(middleware ...HandlerFunc) IRoutes {
    group.Handlers = append(group.Handlers, middleware...)
    return group.returnObj()
}
```

不过, 除了限制 handlers 的总数以外, 没看懂这个 copy 来 copy 去, 跟直接 append 有啥区别 = 。 =

-----------

所以, 说了半天, 其实在 **Gin** 里边 **middleware** 就是 `HandlerFunc`.

比如 `admin.Use(gin.Logger())`, **Logger** 函数的返回值就是 **HandlerFunc**, 即签名为 `func(*Context)` 的函数类型

```go
// Logger instances a Logger middleware that will write the logs to gin.DefaultWriter.
// By default gin.DefaultWriter = os.Stdout.
func Logger() HandlerFunc {
    return LoggerWithConfig(LoggerConfig{})
}
```

同样的, 当我们要**自定义**一些 middleware 的时候, 不管处理的逻辑怎么着, 最后都是返回一个有且仅有一个 `*gin.Context` 参数, 且没有返回值的函数类型.

这里先看下如果不是使用 Gin 框架, 只是 `net/http` 的话, 实现类似 middleware 的功能.

```go
package main

import (
    "log"
    "net/http"
    "time"
)

func main() {
    http.Handle("/", customMiddleware(&MyHandler{}))
    http.Handle("/ping", customMiddleware(http.HandlerFunc(hello)))
    http.ListenAndServe(":8080", nil)
}

func hello(w http.ResponseWriter, r *http.Request) {
    log.Println("log hello")
    w.Write([]byte("pong"))
}

type MyHandler struct{}
func (m *MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    log.Println("log in MyHandler")
    w.Write([]byte("MyHandler ServeHTTP"))
}

func customMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Println(time.Now())
        next.ServeHTTP(w, r)
        log.Println("after: next.ServeHTTP(w, r)")
    })
}
```

这个输入参数 `http.Handler`, 返回值也是 `http.Handler` 的函数就完成了一个简(mei)单(yong)的 middleware 编写了. 其中输入的参数 `http.Handler` 则是下一层要执行的操作.

然后, Gin 主页的 README 就有一个自定义 middleware 例子直接拿来了:

```go
package main

import (
    "github.com/gin-gonic/gin"
    "log"
    "time"
)

func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        t := time.Now()
        // Set example variable
        c.Set("example", "12345")
        // before request

        // after request
        latency := time.Since(t)
        log.Print(latency)
        // access the status we are sending
        status := c.Writer.Status()
        log.Println(status)
    }
}

func main() {
    r := gin.New()
    r.Use(Logger())
    r.GET("/test", func(c *gin.Context) {
        example := c.MustGet("example").(string)
        // it would print: "12345"
        log.Println(example)
    })
    // Listen and serve on 0.0.0.0:8080
    r.Run(":8080")
}
```

对比下来, 不管是 **net/http** 的实现还是 **Gin** 的实现其实都是

> 不断地进行函数压栈再出栈

但因为有了 **gin.Context** 的封装和串联, 所以, 写起来要好看一些.

---------

所以, 最后调用的时候, 关键点在 `func (c *Context) Next()`, **Context** 的 index 默认为 -1,

```go
func (engine *Engine) handleHTTPRequest(c *Context) {
    httpMethod := c.Request.Method
    path := c.Request.URL.Path
    // ...
    // Find root of the tree for the given HTTP method
    t := engine.trees
    for i, tl := 0, len(t); i < tl; i++ {
        // ...
        if handlers != nil {
            c.handlers = handlers
            c.Params = params
            c.Next()
            c.writermem.WriteHeaderNow()
            return
        }
        // ...
    }

    // ...
}

func (c *Context) Next() {
    c.index++
    for s := int8(len(c.handlers)); c.index < s; c.index++ {
        c.handlers[c.index](c)
    }
}
```

------------

Related:

- [从 net/http 入门到 Gin 源码梳理](https://xguox.me/gin-source-code.html)
- [https://chai2010.gitbooks.io/advanced-go-programming-book/content/](https://chai2010.gitbooks.io/advanced-go-programming-book/content/)
- [https://stackoverflow.com/questions/49668070/how-does-servehttp-work](https://stackoverflow.com/a/49674486/1251496)

