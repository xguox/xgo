---
title: "用 Goland 阅读代码是真 6"
date: 2019-01-07T16:01:23+08:00
draft: false
tags: ["Go", "Tools"]
categories: ["Go", "Tools"]
---

曾几何时也不喜欢 IDE, 启动超慢, 风扇呼呼的转. 写 Ruby 以后更甚, 因为 Ruby / Rails 的很多魔法在 IDE 里边会水土不服.

虽然据说 **Rubymine** 挺好的, 不过尝试的欲望还是不高.

---------

之前写 Go 基本都用的是 **Visual Studio Code**, 很多 IDE 的功能基本都能完成. 启动虽然可能不及 **Sublime**, **Vim**(其实装上一堆插件也启动的挺慢的), 不过开箱即用(装个官方的 Go 插件)也不折腾了.

只是 go 1.11 以后对 **go modules** 支持似乎不太好(还要额外自己手工 go get 一些包). 于是开始尝试了 **Goland**, 写的时候虽然有很多魔法快捷方式支持, 但是可能还是更熟悉编辑器的那套 = 。 =

但是, 阅读代码可谓甩编辑器几条街. **Go To Declaration** 本来就是 IDE 的强项了, 虽然 Visual Studio Code 也能做到, 但是, **速度**和**精准度**就只能用差强人意形容. 用了 Goland 定义的跳转直接起飞了.

**Goland** 还有个至少目前没看到在其他一些编辑器上被实现的强大功能, 就是可以**一键查看到接口被哪些类型实现以及实现了哪些接口.**

![](http://wx3.sinaimg.cn/large/62fdd4d5gy1fz0pwagyozj21xg0x6tmp.jpg)