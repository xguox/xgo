---
title: "解决 Lightroom 闪退"
date: 2013-11-28T16:01:23+08:00
draft: false
tags: ["Photo"]
categories: ["Photo"]
---

接[上一篇](http://xguox.me/lightroom-presets.html), 想着在网上搞了那么多的预设,不用放着也不碍事. 结果没些时间, Lightroom 就各种闪退各种编辑不能.

按理说应该不是跟 Maverick 兼容问题吧,如果那样的话升级了之后应该就会有问题了, 不过升级 10.9 以后没怎么使用, 都是随手的就预设搞定. 没注意预设这点, 只是简单地用 CleanMyMac 卸载了之后重装了一次, 顺手下载了一个 5.3 RC版的来解 A7R 的 RAW, 好吧,我也没怎么用 RAW.

安装过后还是老样子的只要一编辑图片就闪退, 蹦出一个 Error 框来唬人(可以进入 Lightroom,看到图片). 不过奇怪的是, 我用 CleanMyMac 卸载完重装以后, 居然还可以继续使用之前的预设. 打开 `~/Library/Application Support/Adobe/Lightroom/Develop Presets/` 发现, 一个也没少啊. 这个就奇葩了.

果断把所有预设删除, 杠杠的就可以编辑!!!果然, 不过是预设太多了还是因为部分预设的版本太老或者其他什么的原因?

把之前的预设都删光光了之后一个个预设文件夹的复制进去,发现,有些导致了不能编辑, 有些则直接就造成 Lightroom 启动不能. 更具体原因不明...

所以说,还是别贪心啊啊, 找好自己喜欢的风格喜欢的调子就好. 这下不能拖延症了啊!