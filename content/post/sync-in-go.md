---
title: "sync.Mutex, sync.RWMutex, snyc.Cond"
date: 2018-11-21T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

当两个(或以上)的 goroutine 并发访问同一个变量, 且至少其中一个是写操作的时候就会发生数据竞争. 像其他语言比如 Ruby 一样, Go 也提供了互斥锁 Mutex 来避免发生这一情况.

```go
package main

import (
	"fmt"
	"sync"
)

var (
	counter int
	wg      sync.WaitGroup
	mutex   sync.Mutex
)

func main() {
	wg.Add(2)

	go incCounter(1)
	go incCounter(2)

	wg.Wait()
	fmt.Printf("Final Counter: %d\\n", counter)
}

func incCounter(id int) {
	defer wg.Done()

	for count := 0; count < 2; count++ {
		mutex.Lock()
		{
			value := counter
			value++
			counter = value
		}
		mutex.Unlock()
	}
}
```

一个互斥锁可以被用来保护**一个临界区或者一组临界区**. 有了互斥锁, 在同一时刻只有一个 goroutine 进入到该临界区里面执行. 每当有 goroutine 想进入临界区时, 都需要先通过 **Lock** 将互斥锁进行锁定, 每个 goroutine 离开临界区时, 都要立即通过 **Unlock** 进行解锁.  当临界区比较精简的时候可能不会忘了解锁, 但是, 当临界区比较复杂的时候, 比如出现分叉或者提前返回, 往往容易忘记解锁, 这时候可以使用 `defer mutext.Unlock()`,  临界区会延伸到函数作用域的最后一行, 当函数返回甚至发生 **panic** 以后用 **recover** 恢复都会执行解锁.

-------------

因为造成竞争的一个原因是**同时写数据**, 这也就意味着, 如果只是并发读的话是不会发生竞争的. 所以, go 提供了更细粒度的读写锁 **RWMutex**. 顾名思义, 就是把读锁跟写锁分开,  通过 **RLock** 与 **RUnlock** 对读锁进行锁定与解锁, 而写锁的锁定与解锁操作则还是**Lock** 和 **Unlock**. 在读锁被锁定的情况下, 如果锁定写锁则会阻塞当前 goroutine, 但是, 再对读锁锁定不会阻塞.

就是说, 多个读操作可以同时进行, 但当有正在读的操作发生以后, 不能进行写操作, 直到读锁被释放.

看了 RWMutex 的源码这里又得回顾一下类型别名和类型定义 = 。 =

```go
// RLocker returns a Locker interface that implements
// the Lock and Unlock methods by calling rw.RLock and rw.RUnlock.
func (rw *RWMutex) RLocker() Locker {
	return (*rlocker)(rw)
}

type rlocker RWMutex

func (r *rlocker) Lock()   { (*RWMutex)(r).RLock() }
func (r *rlocker) Unlock() { (*RWMutex)(r).RUnlock() }
```

配合 sync.Cond 使用的时候, 如果需要传读锁的话可以用这个 `RLocker`

-------------

**snyc.Cond** 需要配合锁一起使用, *Mutex 或者 *RWMutex 都可以. 通过 `sync.NewCond` 函数初始化得到一个 *sync.Cond 值.

```go
// NewCond returns a new Cond with Locker l.
func NewCond(l Locker) *Cond {
	return &Cond{L: l}
}
```

*sync.Cond 的主要方法有三个, `Wait`, `Signal`, `Broadcast`

```go
type Cond struct {
	noCopy noCopy

	// L is held while observing or changing the condition
	L Locker

	notify  notifyList
	checker copyChecker
}
...
```

调用 `newCond` 的时候传的一般是锁的指针, 所以, 下面的 c.L.Lock() 其实跟直接在锁上面调 `Lock()` 是一样的.

```go
//    c.L.Lock()
//    for !condition() {
//        c.Wait()
//    }
//    ... make use of condition ...
//    c.L.Unlock()
//
func (c *Cond) Wait() {
	c.checker.check()
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}
```

`Wait` 会自动 Unlock 锁, 所以, 得先锁定了互斥锁的前提下才能调用 `Wait`. 同时还会暂停执行当前 goroutine, 当稍后唤醒通知来了, 恢复该 goroutine 的执行以后, 会再次上锁.

源码的注释示例这里用的是 **for**, 而不是 **if**, 因为在恢复执行了以后, **condition 不一定就满足条件可以跳出等待**, 那么就需要继续等待, 而不是让 goroutine 执行下去.

对应上面的唤醒通知的是, `Signal`, `Broadcast`这俩方法, Signal 唤醒的任意一个被悬起的 goroutine(Cond 的 `notifyList`), Broadcast 则是唤醒所有的 goroutine, 再分别检验条件是否继续 `Wait`.
