---
title: "Time Machine & Local Snapshots"
date: 2013-11-12T16:01:23+08:00
draft: false
tags: []
categories: []
---

因为 Air 的硬盘只有 120G,所以总习惯性地
 -  关于本机 - 更多信息

然后每次都看着那蛋疼的数字,  然后这次略离谱了

![](http://ww4.sinaimg.cn/mw690/62fdd4d5gw1eaga1xtdgyj20g0057q3c.jpg)

有一段时间没 check, 话说之前一般都维持在 30+ G 的呀,奇怪了那个备份是什么玩意, 我明明已经设置了 Time Machine 备份到我外接的移动硬盘了并且基本上一直都连着这个盘的呀!  查询之,才发现貌似还有个叫 [Local Snapshots(本地快照)](http://support.apple.com/kb/HT4878?viewlocale=zh_CN) 的功能(?).

艾玛, 这么个备份法我的硬盘要蛋碎了. 但是这玩意是备份在哪了的说? 再查之, 原来是在
`/Volumes`

果断删之,  虽然数据的确重要,但是 已经有个外接盘24小时一次的备份了,除非遇上移动盘跟系统同时崩了.

另外据说可以直接
`sudo tmutil disablelocal` 关闭本地快照,
或者
`sudo tmutil enablelocal` 再启动之.

想起前不久[苹果官方](http://www.apple.com/hk/en/support/macbookair-flashdrive/)召回2012款 Macbook Air 的事. 貌似那一款的东芝盘有什么重大 bug, 艾玛, 刚好就是我这款, 而且我的这个盘也是东芝的, 如果配的是三星的SSD貌似没事.
不过其实也一直没问题, 官方说装了个补丁没事就好偶也没啥好不乐意的.  就算无故崩了也还有那高频率的备份.

PS. 影片 + 照片好久没整理了,一直滞留着, 又是一笔大开销, 倒是那个其他还是那么神奇,一会大一会小.
