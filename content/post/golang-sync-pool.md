---
title: "sync.Pool"
date: 2018-12-05T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

```go
package main

import (
	"bytes"
	"fmt"
	"runtime"
	"sync"
)

func main() {
	var pool = &sync.Pool{
		New: func() interface{} {
			return new(bytes.Buffer)
		}}
	b := pool.Get().(*bytes.Buffer)
	b.Write([]byte("hello"))
	pool.Put(b)
	b = pool.Get().(*bytes.Buffer)
	fmt.Println(b)                          // hello
	fmt.Println(pool.Get().(*bytes.Buffer)) // 再去 Get 什么也没有, 因为只有一个, 且前一条语句已经 Get 了又没 Put 回去

	b.Write([]byte("42082"))
	pool.Put(b)
	fmt.Println(pool.Get().(*bytes.Buffer))
	b.Write([]byte("42039"))
	pool.Put(b)
	runtime.GC()                            // 手动 GC
	fmt.Println(pool.Get().(*bytes.Buffer)) // 已被回收, 啥也没输出

	c := pool.Get().(*bytes.Buffer)
	c.Write([]byte("2nd hello"))
	d := pool.Get().(*bytes.Buffer)
	d.Write([]byte("3rd hello"))

	pool.Put(c)
	pool.Put(d)

	fmt.Println(pool.Get().(*bytes.Buffer)) // 2nd hello
	fmt.Println(pool.Get().(*bytes.Buffer)) // 3rd hello
}

```

对比用 `channel` 实现类似 `sync.Pool` 的 benchmark

```go
// x_test.go
package main

import (
	"sync"
	"testing"
)

type ChanPool chan interface{}

type A struct{}

var syncPool = sync.Pool{
	New: func() interface{} { return new(A) },
}

var chanPool ChanPool = make(chan interface{}, 100)

func get() interface{} {
	select {
	case e := <-chanPool:
		return e
	default:
		return new(A)
	}
}

func put(a interface{}) {
	select {
	case chanPool <- a:
	default:
	}
	return
}

func BenchmarkPool(b *testing.B) {
	for i := 0; i < 20; i++ {
		syncPool.Put(new(A))
	}
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		syncPool.Put(syncPool.Get())
	}
}

func BenchmarkBenchmarkChanPool(b *testing.B) {
	for i := 0; i < 20; i++ {
		put(new(A))
	}
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		a := get()
		put(a)
	}
}

// goos: darwin
// goarch: amd64
// pkg: test
// BenchmarkPool-8                	100000000	        22.5 ns/op	       0 B/op	       0 allocs/op
// BenchmarkBenchmarkChanPool-8   	20000000	        78.4 ns/op	       0 B/op	       0 allocs/op
// PASS
// coverage: 0.0% of statements
// ok  	test	3.941s
// Success: Benchmarks passed.
```