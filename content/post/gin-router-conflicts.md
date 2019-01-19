---
title: "Gin 路由冲突"
date: 2018-12-29T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

[Gin](https://github.com/gin-gonic/gin) 或者说 [httprouter](https://github.com/julienschmidt/httprouter) 的路由使用的数据结构是动态压缩的 [trie](https://en.wikipedia.org/wiki/Trie#Compressing_tries).

![](https://chai2010.cn/advanced-go-programming-book/images/ch6-02-radix-get-4.png)

每个 HTTP 方法维护着一个 trie. 不像 Rails 遇到路由冲突的时候, 定义在前的会覆盖后面的, Gin 在 build 的时候就会 panic.

几种冲突的场景:

- 在插入 **wildcard** 节点时, 父节点的 children 数组非空且 **wildChild** 被设置为 false. 例如: `GET /user/getAll` 和 `GET /user/:id/getAddr`, 或者 `GET /user/*aaa` 和 `GET /user/:id`.
- 在插入 **wildcard** 节点时, 父节点的 **children** 数组非空且 **wildChild** 被设置为 **true** , 但该父节点的 **wildcard** 子节点要插入的 **wildcard** 名字不一样. 例如: `GET /user/:id/info` 和 `GET /user/:name/info`.
- 在插入 **catchAll** 节点时, 父节点的 **children** 非空. 例如: `GET /src/abc` 和 `GET /src/*filename`, 或者 `GET /src/:id` 和 `GET /src/*filename`.
- 在插入 **static** 节点时, 父节点的 **wildChild** 字段被设置为 **true**.
- 在插入 **static** 节点时, 父节点的 **children** 非空, 且子节点 **nType** 为 **catchAll**.

-------------

Related:

- [https://github.com/chai2010/advanced-go-programming-book](https://github.com/chai2010/advanced-go-programming-book)