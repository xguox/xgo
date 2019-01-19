---
title: "@import vs require"
date: 2014-07-01T16:01:23+08:00
draft: false
tags: ["CSS"]
categories: ["CSS"]
---

因为项目是我中途参入进来的, 所以并没有注意到 CSS(Sass) 的组织一直都是用的 Sprockets 的方式去 require, 尽管这期间写了不少的CSS(Sass), 包括后来当添加一些个变量之后才发现不对劲,  老是报错 Sass 里头变量 undefined.

![](http://ww4.sinaimg.cn/mw690/62fdd4d5gw1eikut0ruixj210c0bstba.jpg)

才知道原来使用链式调用的话,  variables or mixins 的作用域只在其定义的文件内.

而使用 @import 的话则是创建了一个全局的作用域. Guides 看得不够仔细啊!