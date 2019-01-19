---
title: "理解 Goroutine 的调度"
date: 2018-12-21T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

最开始学的时候看的一本书, 当时只是通读看语法的阶段, 其中有几个像下面这样关于 goroutine 调度的图并没有什么概念, 那个章节是关于 Go 的并发的. 现在回头理一理这一部分的知识.

![](http://wx2.sinaimg.cn/large/62fdd4d5gy1fyb9awfnvcj22801e07hy.jpg)

![](http://wx1.sinaimg.cn/large/62fdd4d5gy1fyb9bf7cpzj22801e0dom.jpg)

goroutine 是构建在操作系统的线程调度之上的,
图中的主要部分分成三类, 也就是大家每每提及 goroutine 时候会牵扯到的 MPG 调度模式.

- M: 系统级线程
- P: 逻辑处理器
- G: goroutine

**M** 是操作系统线程在 Go runtime 的体现. 操作系统不管 P, G, 调度什么的, 只对接 M. 然后, 就像操作系统需要把线程放到一个 CPU 核心上运行一样, M 也需要绑定到一个 P 上才会运行 G. 按需创建, 默认最大限制为10000个.

**P** for processor, 执行 G 的所必须的资源. P 的数量可以通过调用 **runtime 的 `func GOMAXPROCS(n int) int`** 来设置, 默认为机器的逻辑 CPU 核数(比如四核可能用超线程虚拟成八核的跑), 可以通过 **runtime 的 `func NumCPU() int`** 获取得到值. 最多也只能是逻辑 CPU 核数个线程同时在操作系统跑, 所以 P 的数量决定了最多有多少个 G 可以同时运行. 一般也没必要去修改.

一般情况下调度器会让 M 完整执行 G 的代码, 但是, 遇到下面几种场景时候会切换到其他的 G

- G 里边有新的 G 产生, 也就是调用了 `go` 函数
- channel 操作
- 系统调用, 如文件读写
- 网络 I/O
- 一个完整的 GC 周期以后?

---------------

![](http://wx3.sinaimg.cn/large/62fdd4d5gy1fyday1v2bjj20b40avglr.jpg)

图片来源 - [http://morsmachine.dk/go-scheduler](http://morsmachine.dk/go-scheduler)

关于 MPG 这部分相关的 go 源代码基本上就在 `runtime` 包的 **runtime2.go** 和 **proc.go**可以看到. 下面只是截取几个主要的结构体(其实还没完整看明白这些结构体字段的意思 = 。 =):

```go
type g struct {
    stack       stack   // offset known to runtime/cgo
    stackguard0 uintptr // offset known to liblink
    stackguard1 uintptr // offset known to liblink
    _panic         *_panic // innermost panic - offset known to liblink
    _defer         *_defer // innermost defer
    m              *m      // current m; offset known to arm liblink
    sched          gobuf
    ...
}
type m struct {
    g0      *g     // goroutine with scheduling stack
    ...
    curg          *g       // current running goroutine
    p             puintptr // attached p for executing go code (nil if not executing go code)
    ...
    spinning      bool // m is out of work and is actively looking for work
    blocked       bool // m is blocked on a note
    inwb          bool // m is executing a write barrier
    ...
}
type p struct {
    lock mutex
    id          int32
    status      uint32 // one of pidle/prunning/...
    link        puintptr
    schedtick   uint32     // incremented on every scheduler call
    syscalltick uint32     // incremented on every system call
    sysmontick  sysmontick // last tick observed by sysmon
    m           muintptr   // back-link to associated m (nil if idle)
    ...
    // Queue of runnable goroutines. Accessed without lock.
    runqhead uint32
    runqtail uint32
    runq     [256]guintptr
}
type schedt struct {
    // Global runnable queue.
    runqhead guintptr
    runqtail guintptr
    runqsize int32
}
```

调度器 `schedt` 结构体维护着一个全局的 goroutines 队列, 然后
每个 P 都管理着一组本地 goroutines 队列 **runq**, 每当有 `go` 函数调用产生(`func newproc(siz int32, fn *funcval)`)一个 goroutine 就会被加入到全局/本地队列中, 具体规则没理解清楚 (Put it on the queue of g's waiting to run). 查了其他文档貌似说是优先放入当前 P 的本地队列中.

```go
// Create a new g running fn with siz bytes of arguments.
// Put it on the queue of g's waiting to run.
// The compiler turns a go statement into a call to this.
// Cannot split the stack because it assumes that the arguments
// are available sequentially after &fn; they would not be
// copied if a stack split occurred.
//go:nosplit
func newproc(siz int32, fn *funcval) {
    argp := add(unsafe.Pointer(&fn), sys.PtrSize)
    gp := getg()
    pc := getcallerpc()
    systemstack(func() {
        newproc1(fn, (*uint8)(argp), siz, gp, pc)
    })
}
```

当 M 执行完了当前 P 的 本地队列中所有的 G 以后, 这个 P 会先尝试从全局队列中拿一些 G 来执行, 当全局队列为空的时候, 就会随机挑选其他的 P, 从这个随机 P 的本地队列里中拿走一半的 G 到自己的队列中执行. ( **proc.go** 的 `func runqsteal(_p_, p2 *p, stealRunNextG bool) *g` )

![](http://wx1.sinaimg.cn/large/62fdd4d5gy1fyday2bzmnj20fa0b40sx.jpg)

---------------

![](http://wx1.sinaimg.cn/large/62fdd4d5gy1fyday2naj5j20fa0b474h.jpg)

当某个 G 遇到前面提到的几种阻塞或者可能需要切换上下文的场景时, 调度器会让 P 会与 M(0) 分离, 并创建一个新的 M(1) 或者从缓存起来的其他 M(1) 启动来运行这个 P 上其余的 G, 而 M(0)继续运行着这个阻塞的 G. 当阻塞 G 执行完成以后会被放回 runq 队列, 看其他文章有说放回全局队列, 也有说放回原本的队列 = 。 = 反正就是放回队列, 然后 M(0) 可以洗洗睡了.

---------------

Related:

- Go: Design Patterns for Real-World Projects
- Go in Action
- [https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)
- [https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/](https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/)
- [https://golang.org/s/go11sched](https://golang.org/s/go11sched)
