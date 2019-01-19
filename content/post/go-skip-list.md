---
title: "Go 实现 Skip List(跳表)"
date: 2018-11-10T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

![](http://wx2.sinaimg.cn/large/62fdd4d5gy1fwzsypsodkj20ti026wee.jpg)

**在链表中查找数据的时间复杂度是 O(n)**, 在上面这个链表, 假设要查找到节点 37, 就得从第一个节点开始, 遍历 7 次,

![](http://wx3.sinaimg.cn/large/62fdd4d5gy1fwzsyou92cj20ti03at8p.jpg)

如果通过给链表节点加一层索引, 每两个节点提取出来一个, 组成一个新的链表, 抽取出来的每一个节点, 除了原本有的指向原始链表(最下面一层)的下一节点的指针外, 还有另一个指针, 指向新一级的链表中对应的下一个节点, 通过这个跳表结构, 还是查找节点 37, 就只需要遍历 4 次 (7 ->> 19 ->> 26 -> 37).

![](http://wx2.sinaimg.cn/large/62fdd4d5gy1fwzsyo73cmj20ti04xq2z.jpg)

以此类推, 在上面这个提取出来的新一级链表之上, 再一次提取出来一级新的链表. 依旧同样查找节点 37 ,  现在只要遍历 3 次.

类似二分查找的, 每提取一级链表, 节点数都是前一级的一半, 所以, 查找的时间复杂度就从**原始链表的 O(n) 降低到 O(log n)**

那么, 问题来了, 每次新增或者删除一个节点, 就会打乱原本的跳表结构节点数的比例, 极端情况下, 可能最后又变回一条巨大的链表.

实际上, **跳表并不要求上下相邻两层链表之间的节点个数有严格的对应关系**, 当插入数据时候, 通过生成一个随机整数来决定这个节点会被插入到哪几层链表中, 比如, 生成的随机数是 K, 那么这个结点就插入到从第 1 到 第 K 层链表中(以及原始链表), 节点最大的层数不允许超过一个特定的最大值 MaxLevel,  而插入操作不影响其他节点的层数. 删除同理.

> skiplist, 翻译成中文, 可以翻译成"跳表"或"跳跃表", 指的就是除了最下面第1层链表之外, 它会产生若干层稀疏的链表, 这些链表里面的指针故意跳过了一些节点(而且越高层的链表跳过的节点越多). 这就使得我们在查找数据的时候能够先在高层的链表中进行查找, 然后逐层降低, 最终降到第1层链表来精确地确定数据位置. 在这个过程中, 我们跳过了一些节点, 从而也就加快了查找速度.

![](http://wx4.sinaimg.cn/large/62fdd4d5gy1fwzsyp9rosj20vp116dib.jpg)

或者看这个维基百科给的 gif

![](http://wx1.sinaimg.cn/large/62fdd4d5gy1fwyefapcpzg20qo098t9r.gif)

相关来源:

- [https://en.wikipedia.org/wiki/File:Skip_list_add_element-en.gif](https://en.wikipedia.org/wiki/File:Skip_list_add_element-en.gif)
- [https://commons.wikimedia.org/wiki/File:Skip_list.svg](https://commons.wikimedia.org/wiki/File:Skip_list.svg)
- [http://zhangtielei.com/posts/blog-redis-skiplist.html](http://zhangtielei.com/posts/blog-redis-skiplist.html)

--------

##### 用 Go 实现跳表

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

const MaxLevel = 32
const Probability = 0.25 // 基于时间与空间综合 best practice 值, 越上层概率越小

func randLevel() (level int) {
    rand.Seed(time.Now().UnixNano())
    for level = 1; rand.Float32() < Probability && level < MaxLevel; level++ {
        // fmt.Println(rand.Float32())
    }
    // fmt.Printf("up to %d level\n", level)
    return
}

type node struct {
    forward []*node
    key     int
}

type skipList struct {
    head  *node
    level int
}

func newNode(key, level int) *node {
    return &node{key: key, forward: make([]*node, level)}
}

func newSkipList() *skipList {
    return &skipList{head: newNode(0, MaxLevel), level: 1}
}

func (s *skipList) insert(key int) {
    current := s.head
    update := make([]*node, MaxLevel) // 新节点插入以后的前驱节点
    for i := s.level - 1; i >= 0; i-- {
        if current.forward[i] == nil || current.forward[i].key > key {
            update[i] = current
        } else {
            for current.forward[i] != nil && current.forward[i].key < key {
                current = current.forward[i] // 指针往前推进
            }
            update[i] = current
        }
    }

    level := randLevel()
    if level > s.level {
        // 新节点层数大于跳表当前层数时候, 现有层数 + 1 的 head 指向新节点
        for i := s.level; i < level; i++ {
            update[i] = s.head
        }
        s.level = level
    }
    node := newNode(key, level)
    for i := 0; i < level; i++ {
        node.forward[i] = update[i].forward[i]
        update[i].forward[i] = node
    }
}

func (s *skipList) delete(key int) {
    current := s.head
    for i := s.level - 1; i >= 0; i-- {
        for current.forward[i] != nil {
            if current.forward[i].key == key {
                tmp := current.forward[i]
                current.forward[i] = tmp.forward[i]
                tmp.forward[i] = nil
            } else if current.forward[i].key > key {
                break
            } else {
                current = current.forward[i]
            }
        }
    }

}

func (s *skipList) search(key int) *node {
    // 类似 delete
    return nil
}

func (s *skipList) print() {
    fmt.Println()

    for i := s.level - 1; i >= 0; i-- {
        current := s.head
        for current.forward[i] != nil {
            fmt.Printf("%d ", current.forward[i].key)
            current = current.forward[i]
        }
        fmt.Printf("***************** Level %d \n", i+1)
    }
}

func main() {
    list := newSkipList()
    for i := 0; i < 20; i++ {
        list.insert(rand.Intn(100))
    }
    list.print()

    fmt.Println("\n--------------------------------------")

    list.delete(10)
    list.print()

    fmt.Println("\n--------------------------------------")
}

```