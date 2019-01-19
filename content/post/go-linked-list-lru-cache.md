---
title: "Golang, 链表, LRU 缓存淘汰策略"
date: 2018-10-15T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

当缓存的空间即将达到临界值的时候, 需要将一些旧的数据清理掉, 哪些该去, 哪些该留, 常用的缓存淘汰策略有下面三种:

- FIFO(First In，First Out) 先进先出策略
- LFU(Least Frequently Used) 最少使用策略
- LRU(Least Recently Used) 最近最少使用策略

这里**基于 Go 分别使用单向链表(Singly Linked List)和双向链表(Doubly Linked List)**实现最后的这个 **LRU 最近最少使用策略**.

各种术语加上英文缩写看着好像很流弊, 用人话描述这个策略的基本思路其实也不难理解:

1. **新的数据插入到链表头部**
2. **当链表中的数据被访问以后, 将该数据重置到链表头部**
3. **当链表达到临界值时候, 将链表尾部的数据丢弃**

##### 单向链表(Singly Linked List)

```go
package main

import "fmt"

type SinglyLink struct {
    head     *Node
    capacity int
    size     int
}

type Node struct {
    Value int
    Next  *Node
}

func (s *SinglyLink) prependNode(val int) {
    currentHead := s.head
    s.size++
    if currentHead == nil {
        s.head = &Node{val, nil}
        return
    }

    s.head = &Node{val, currentHead}
}

func (s *SinglyLink) deleteLastNode() {
    if s.size <= 1 {
        s.head = nil
        s.size = 0
        return
    }

    tmp := s.head

    for tmp.Next.Next != nil {
        tmp = tmp.Next
    }
    tmp.Next = nil
    s.size--
}

func (s *SinglyLink) deleteNode(val int) {
    if s.size == 0 {
        return
    }
    currentNode := s.head

    if currentNode.Value == val {
        s.head = currentNode.Next
        s.size--
        return
    }
    for currentNode.Next != nil {
        if currentNode.Next.Value == val {
            currentNode.Next = currentNode.Next.Next
            s.size--
            return
        }
        currentNode = currentNode.Next
    }
}

func (s *SinglyLink) searchNode(val int) {
    currentNode := s.head
    for currentNode != nil {
        if currentNode.Value == val {
            s.deleteNode(val)
            s.prependNode(val)
            return
        }
        nextNode := currentNode.Next
        if nextNode == nil {
            s.checkFull(val)
            return
        }
        currentNode = nextNode
    }
}

func (s *SinglyLink) checkFull(val int) {
    isFull := s.size >= s.capacity
    if isFull {
        s.deleteLastNode()
    }
    s.prependNode(val)
}

func (s *SinglyLink) print() {
    tmp := s.head
    for tmp != nil {
        fmt.Printf("%d -> ", tmp.Value)
        tmp = tmp.Next
    }
    fmt.Println()
}

func newSinglyLink(capacity int) *SinglyLink {
    return &SinglyLink{nil, capacity, 0}
}

func main() {
    lru := newSinglyLink(3)
    lru.prependNode(1)
    lru.prependNode(0)
    lru.prependNode(2)
    lru.searchNode(7)
    lru.print() // 7 -> 2 -> 0 ->
    lru.searchNode(4)
    lru.print() // 4 -> 7 -> 2 ->
    lru.searchNode(22222)
    lru.searchNode(2)
    lru.searchNode(99)
    lru.print() // 99 -> 2 -> 22222 ->
}

```

实际上, 也不会有人只用一个单向链表来实现 LRU (练习用 Go 写链表 = 。 = ), 插入和删除**本身**的时间复杂度虽然都是 O(1), 但是, 如果要插入或者删除特定位置的节点, 还得遍历查找, 时间复杂度 O(n). 所以, 一般都会连同使用哈希表来优化查找.

##### 双向链表(Doubly Linked List)

事实是, [Go 已经内建有双向链表](https://golang.org/src/container/list/list.go), 所以, 直接用就是了. 😜

```go


package main

import (
	"container/list"
	"fmt"
)

type LruCache struct {
	capacity int
	items    map[string]*list.Element
	link     *list.List
}

type Item struct {
	key   string
	value interface{}
}

func NewLruCache(capacity int) *LruCache {
	return &LruCache{
		capacity: capacity,
		items:    make(map[string]*list.Element),
		link:     list.New(),
	}
}

func (l *LruCache) Get(key string) (value interface{}, exists bool) {
	if item, exists := l.items[key]; exists {
		l.link.MoveToFront(item) // moves element e to the front of list l. If e is not an element of l, the list is not modified. The element must not be nil.
		value = item.Value.(*Item).value
	}
	return
}

func (l *LruCache) Set(key string, value interface{}) {
	if item, exists := l.items[key]; exists {
		// 移到头结点并更新值
		l.link.MoveToFront(item)
		item.Value.(*Item).value = value
		return
	}
	item := l.link.PushFront(&Item{key, value}) // inserts a new element at the front of list
	l.items[key] = item
	if l.link.Len() > l.capacity {
		l.DeleteOldest()
	}
}

func (l *LruCache) DeleteOldest() {
	item := l.link.Back() // returns the last element of list or nil if the list is empty.
	if item != nil {
		l.removeItem(item)
	}
}

func (l *LruCache) delete(key string) {
	if item, exists := l.items[key]; exists {
		l.removeItem(item)
	}
}

func (l *LruCache) removeItem(el *list.Element) {
	l.link.Remove(el) // removes el from list if el is an element of the list. It returns the element value el.Value. The element must not be nil.

	item := el.Value.(*Item)
	delete(l.items, item.key)
}

func (l *LruCache) print() {
	tmp := l.link.Front() // returns the first element of list or nil if the list is empty.
	for tmp != nil {
		fmt.Printf("%v -> ", tmp.Value)
		tmp = tmp.Next() // returns the next list element
	}
	fmt.Println()
}

func main() {
	lru := NewLruCache(5)
	lru.Get("s")
	lru.print()
	lru.Set("o", 90)
	lru.Set("n", 200)
	lru.Set("y", "fujifilm")
	lru.Get("n")
	lru.print() // &{n 200} -> &{y fujifilm} -> &{o 90} ->
	lru.Set("x", "xt3")
	lru.Set("gfx", "50s")
	lru.Set("sony", "a7r")
	lru.Get("gfx")
	lru.print() // &{gfx 50s} -> &{sony a7r} -> &{x xt3} -> &{n 200} -> &{y fujifilm} ->
}

```

最后, 这里的实现并不是并发安全的 = 。 =