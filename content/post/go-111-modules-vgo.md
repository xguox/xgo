---
title: "Go 1.11 modules"
date: 2018-08-28T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

Go 的包依赖管理, 终于在前几天发布的 **1.11** 有了一个初步官方定数. 再也不用纠结于离不开 **$GOPATH**, 1.11 发布之前试用了一下 vgo, 文档比较少, 踩了一些坑就没深入继续了. 这次正式发布的 1.11 其实就是把 vgo 给[正式合并](https://github.com/golang/go/commit/f7248f05946c1804b5519d0b3eb0db054dc9c5d6)进来了.

在 **$GOPATH** 之外使用 go modules, 如果是现有项目的话可以直接 `go mod init`, 现有项目会根据 `git remote` 自动识别 module 名, 但是新项目的话就会报 `go: cannot determine module path for source directory`, 需要带上 module 名.

![](http://wx4.sinaimg.cn/large/62fdd4d5gy1fupsw96mquj22801e0qdh.jpg)

`$GOPATH/pkg/` 下会多了一个 **mod** 目录, 再也不是像 `go get` 下载到 `$GOPATH/src`

--------------------------------------

##### Related:

[Introduction to Go Modules](https://roberto.selbach.ca/intro-to-go-modules)

[Brad Fitzpatrick](https://twitter.com/bradfitz/status/1033159776526524416)