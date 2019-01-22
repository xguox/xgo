---
title: "Go 程序的性能监控与分析 pprof"
date: 2019-01-22T01:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

Go 开箱就提供了一系列的性能监控 API 以及用于分析的工具, 可以快捷而有效地观察应用各个细节的 CPU 与内存使用概况, 包括生成一些可视化的数据(需要额外安装 [Graphviz](https://www.graphviz.org/)).

[例子 gist](https://gist.github.com/xguox/dcabd3573b39bf64bc7b961d4a64f478) 来自之前的 [Trie 的实现, Ruby vs Go](https://xguox.me/trie-implementing-ruby-vs-golang.html/).

`main` 函数加上了下面几行:

```go
import "runtime/pprof"
// ...
cpuProfile, _ := os.Create("cpu_profile")
pprof.StartCPUProfile(cpuProfile)
defer pprof.StopCPUProfile()
// ...
```

这里 `os.Create("cpu_profile")` 指定生成的数据文件, 然后 `pprof.StartCPUProfile` 看名字就知道是开始对 CPU 的使用进行监控. 有开始就有结束, 一般直接跟着 `defer pprof.StopCPUProfile()` 省的后面忘了. 编译执行一次以后会在目录下生成监控数据并记录到 **cpu_profile**. 接着就可以使用 **pprof** 来解读分析这些监控生成的数据.

> When CPU profiling is enabled, the Go program stops about 100 times per second and records a sample consisting of the program counters on the currently executing goroutine's stack.

# CPU Profiling

```go
$ go tool pprof cpu_profile
Type: cpu
Time: Jan 22, 2019 at 3:02pm (CST)
Duration: 518.52ms, Total samples = 570ms (109.93%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```

因为是在多核环境, 所以, 取样时间(Total samples) 占比大于 100% 也属于正常的. 交互操作模式提供了一大票的命令, 执行一下 `help` 就有相应的文档了. 比如输出报告到各种格式(pdf, png, gif), 方块越大个表示消耗越大.

![](http://wx3.sinaimg.cn/large/62fdd4d5gy1fzfdl1w0vkj21jo1dyh48.jpg)

又或者列出 CPU 占比最高的一些(默认十个)运行结点的 `top` 命令, 也可以加上需要的结点数比如 `top15`

```go
(pprof) top
Showing nodes accounting for 480ms, 84.21% of 570ms total
Showing top 10 nodes out of 67
    flat  flat%   sum%        cum   cum%
    200ms 35.09% 35.09%      210ms 36.84%  main.NewNode (inline)
    70ms 12.28% 47.37%      170ms 29.82%  runtime.scanobject
    60ms 10.53% 57.89%       70ms 12.28%  runtime.greyobject
    30ms  5.26% 63.16%       30ms  5.26%  runtime.memclrNoHeapPointers
    30ms  5.26% 68.42%       30ms  5.26%  runtime.memmove
    20ms  3.51% 71.93%      250ms 43.86%  main.(*Node).insert
    20ms  3.51% 75.44%       20ms  3.51%  runtime.findObject
    20ms  3.51% 78.95%      230ms 40.35%  runtime.gcDrain
    20ms  3.51% 82.46%       20ms  3.51%  runtime.pthread_cond_wait
    10ms  1.75% 84.21%       10ms  1.75%  runtime.(*gcWork).tryGetFast (inline)
```

- **flat**: 是指该函数执行耗时, 程序总耗时 570ms, `main.NewNode` 的 200ms 占了 35.09%
- **sum**: 当前函数与排在它上面的其他函数的 flat 占比总和, 比如 `35.09% + 12.28% = 47.37%`
- **cum**: 是指该函数加上在该函数调用之前累计的总耗时, 这个看图片格式的话会更清晰一些.

可以看到, 这里最耗 CPU 时间的是 `main.NewNode` 这个操作.

除此外还有 `list` 命令可以根据匹配的参数列出指定的函数相关数据, 比如:

![](http://wx4.sinaimg.cn/large/62fdd4d5gy1fzfdmog0otj22801e07er.jpg)

# Memory Profiling

```go
// ...
memProfile, _ := os.Create("mem_profile")
pprof.WriteHeapProfile(memProfile)
```

类似 CPU 的监控, 要监控内存的分配回收使用情况, 只要调用 `pprof.WriteHeapProfile(memProfile)`

![](http://wx2.sinaimg.cn/large/62fdd4d5gy1fzfcrz69vwj22801e0nce.jpg)

然后是跟上面一样的生成图片:

![](http://wx3.sinaimg.cn/large/62fdd4d5gy1fzfcvmtt83j21y617aayy.jpg)

**Type: inuse_space** 是监控内存的默认选项, 还可以选 **-alloc_space**, **-inuse_objects**, **-alloc_objects**

inuse_space 是正在使用的内存大小, alloc_space是从头到尾一共分配了的内存大小(包括已经回收了的), 后缀为 `_objects` 的是相应的对象数

# net/http/pprof

对于 http 服务的监控有一些些的不同, 不过 Go 已经对 pprof 做了一些封装在 `net/http/pprof`

[例子 gist](https://gist.github.com/xguox/f0603d9e3ef48148d4bd84fa209c6de5) 来自[从 net/http 入门到 Gin 源码梳理](https://xguox.me/gin-source-code.html/)

引入多一行 `_ "net/http/pprof"`, 启用服务以后就可以在路径 `/debug/pprof/` 看到相应的监控数据. 类似下面(已经很贴心的把各自的描述信息写在下边了):

![](http://wx1.sinaimg.cn/large/62fdd4d5gy1fzfeeircq4j22801e0akr.jpg)

![](http://wx2.sinaimg.cn/large/62fdd4d5gy1fzfeejcmncj22801e0kar.jpg)

还是没有前面的那些可视化图形 UI 直观, 不过可以通过 **http://localhost:8080/debug/pprof/profile** (其他几个指标也差不多, heap, alloc...)生成一个类似前面的 CPU profile 文件监控 **30s** 内的数据. 然后就可以用 `go tool pprof`来解读了.

## gin pprof

`import _ "net/http/pprof"` 实际上是为了执行包 **net/http/pprof** 中的 init 函数.

```go
// pprof.go
func init() {
    http.HandleFunc("/debug/pprof/", Index)
    http.HandleFunc("/debug/pprof/cmdline", Cmdline)
    http.HandleFunc("/debug/pprof/profile", Profile)
    http.HandleFunc("/debug/pprof/symbol", Symbol)
    http.HandleFunc("/debug/pprof/trace", Trace)
}
```

因此, **Gin** 项目要使用 **pprof** 的话可以参考[这里](https://github.com/DeanThompson/ginpprof/blob/master/pprof.go)
