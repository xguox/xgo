---
title: "css3 之 column-*"
date: 2014-02-13T16:01:23+08:00
draft: false
tags: ["CSS"]
categories: ["CSS"]
---

原本想要使用 CSS3 中的多栏属性实现类似报纸那样的排版, 或者 pinterest 那样的瀑布流,  但结果跟预想相去较远.
原以为用 column-count 或 column-width 来将 HTML 元素变成多栏, 再 column-gap 定义为 0 让所有元素能贴在一起,  但是才发现没有真正理解多栏的含义,  使用多栏的话是从上到下的排序, 而如果按时间顺序列出所有 posts 的话,  那结果只是

基于这样的排序, 于是就想着将计就计,  把页面整体做成横向滚动, 为此还使用了 [jquery.mousewheel.min.js]() 来把鼠标滚动设置成[横向](http://handhack.com/explore).  想来绝大部分的网页都是上下滚动, 搞搞非主流.   最后发现,  整体这么设置的话, 会造成用户观看的时候视线变成了频繁的正弦曲线, 很糟糕.  现在(14.2.14)还是这个样子, 因为没有想到其他 nice 的方案, 暂时没做修改.

主流浏览器对 `column-*`的支持各不相同,
又比如, `moz-column-count` can not work properly with  `-moz-column-fill: auto`, 见 [https://bugzilla.mozilla.org/show_bug.cgi?id=733808](https://bugzilla.mozilla.org/show_bug.cgi?id=733808)
所以像这样整体的过度使用暂时还是放弃为好.