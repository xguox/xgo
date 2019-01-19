---
title: "Go 的指针与数组"
date: 2019-01-04T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

[上一篇](https://xguox.me/trie-implementing-ruby-vs-golang.html) 在用 Go 实现 **Trie** 结构时候踩了个坑,

```go
type Node struct {
    Char     rune
    Children []*Node
}
```

最开始这里 `Children` 字段用的是 `[]Node` 而不是 `[]*Node`, 结果数据只塞进了一层以后, 每一结点的这个 `Children` 都为空的 `[]Node`.

Debug 了好一会一步步的执行下来才发现问题所在, **数组在元素赋值的时候是传副本的.**

改一下 **insert** 方法打印一下内存地址就知道了.

```go
func (n *Node) insert(r rune) *Node {
    child := n.get(r)
    if child == nil {
        child = NewNode(r)
        n.Children = append(n.Children, *child)
        fmt.Printf("new child addr: %p \n", &child)
        last := n.Children[len(n.Children)-1]
        fmt.Printf("but last addr: %p \n", &last)
    }

    return child
}
```

输出结果类似:

```go
...
new child addr: 0xc000287f18
but last addr: 0xc0002a3640
new child addr: 0xc000287f20
but last addr: 0xc0002a36c0
new child addr: 0xc000287f28
but last addr: 0xc0002a3740
```

append 进去和取出来的是不一样的地址 = 。 = 设置进去的只是副本, 类似方法调用的参数一样. (**map**, **channel** 也是)