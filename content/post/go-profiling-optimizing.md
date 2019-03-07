---
title: "Go 程序的性能监控与分析 pprof"
date: 2019-01-23T01:01:23+08:00
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

用 wrk (brew install wrk) 模拟测试

`wrk -c 200 -t 4 -d 3m http://localhost:8080/hello`

![](http://wx3.sinaimg.cn/large/62fdd4d5gy1fzfftiaid1j22801e0akx.jpg)

还是没有前面的那些可视化图形 UI 直观, 不过可以通过 **http://localhost:8080/debug/pprof/profile** (其他几个指标也差不多, heap, alloc...)生成一个类似前面的 CPU profile 文件监控 **30s** 内的数据. 然后就可以用 `go tool pprof`来解读了.

```go
Type: cpu
Time: Jan 22, 2019 at 4:22pm (CST)
Duration: 30.13s, Total samples = 1.62mins (321.66%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 93.78s, 96.76% of 96.92s total
Dropped 270 nodes (cum <= 0.48s)
Showing top 10 nodes out of 52
      flat  flat%   sum%        cum   cum%
    81.42s 84.01% 84.01%     81.45s 84.04%  syscall.Syscall
     3.45s  3.56% 87.57%      3.45s  3.56%  runtime.kevent
     2.31s  2.38% 89.95%      2.31s  2.38%  runtime.pthread_cond_wait
     2.06s  2.13% 92.08%      2.07s  2.14%  runtime.usleep
     1.93s  1.99% 94.07%      1.93s  1.99%  runtime.pthread_cond_signal
     1.10s  1.13% 95.20%      1.10s  1.13%  runtime.freedefer
     0.85s  0.88% 96.08%      0.87s   0.9%  runtime.nanotime
     0.59s  0.61% 96.69%      4.07s  4.20%  runtime.netpoll
     0.04s 0.041% 96.73%      0.67s  0.69%  runtime.newproc1
     0.03s 0.031% 96.76%     44.18s 45.58%  net/http.(*conn).readRequest
(pprof)

Type: alloc_space
Time: Jan 22, 2019 at 4:26pm (CST)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 21.91GB, 99.82% of 21.95GB total
Dropped 66 nodes (cum <= 0.11GB)
Showing top 10 nodes out of 16
      flat  flat%   sum%        cum   cum%
    6.72GB 30.61% 30.61%     6.72GB 30.62%  net/textproto.(*Reader).ReadMIMEHeader
    5.97GB 27.18% 57.80%    20.54GB 93.60%  net/http.(*conn).readRequest
       4GB 18.21% 76.01%    13.23GB 60.30%  net/http.readRequest
    2.01GB  9.16% 85.17%     2.01GB  9.16%  net/url.parse
    1.25GB  5.71% 90.88%     1.25GB  5.71%  net.(*conn).Read
    1.22GB  5.54% 96.42%     1.22GB  5.55%  context.WithCancel
    0.49GB  2.25% 98.68%     0.49GB  2.25%  net/textproto.(*Reader).ReadLine
    0.13GB  0.58% 99.25%     0.13GB  0.58%  main.main.func1
    0.12GB  0.56% 99.82%     0.12GB  0.56%  bufio.NewWriterSize (inline)
         0     0% 99.82%     0.13GB  0.59%  net/http.(*ServeMux).ServeHTTP
(pprof)
```


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

# Flame Graph 火焰图

[go-torch](https://github.com/uber/go-torch) 在 **Go 1.11** 之前是作为非官方的可视化工具存在的, 它可以为监控数据生成一个类似下面这样的图形界面, 红红火火的, 因而得名. 从 **Go 1.11** 开始, 火焰图被集成进入 Go 官方的 **pprof** 库.

![](https://camo.githubusercontent.com/496577fa999cda41f65a01923afc75ed02993845/687474703a2f2f756265722e6769746875622e696f2f676f2d746f7263682f6d6574612e737667)

> go-torch is deprecated, use pprof instead

> As of Go 1.11, flamegraph visualizations are available in go tool pprof directly!

```go
$ go tool pprof -http=":8081" [binary] [profile]
```

在浏览器打开 **http://localhost:8081/ui/flamegraph**, 就可以看到下面这样的反过来的火焰图.

长条形的颜色只是为了好看, 颜色的深浅是随机的 = 。=  长度越长代表占用 CPU 时间越长

![](http://wx2.sinaimg.cn/large/62fdd4d5gy1fzgbuz1x3nj227y0swaj2.jpg)

然后, **pprof** 命令行的 **top** 以及 **list** 正则也可以在这里边完成, 还有 **svg** 图形.

![](http://wx3.sinaimg.cn/large/62fdd4d5gy1fzgbuzpge6j21le0ocq8b.jpg)


