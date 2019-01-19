---
title: "与开发相关的一些 iOS App (通勤时间太长怪我咯?)"
date: 2016-12-12T16:01:23+08:00
draft: false
tags: ["Tools"]
categories: ["Tools"]
---

在路上的时间超长如我, 不是果粉, 只是有一台 iPad Air 2.

这货屏幕比手中的大法机大得多, 平日看电子书较多, 最近折腾起一些开发相关的工具, 好像还是很好玩的.

目测再败个键盘估计会更好使一些. **不是要挑战成为主力机器, 只是可能也许会有用到的时候...**

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1faphivulnyj21kw0bznpd.jpg)

----------

### Termius

SSH client (Free)

最开始因为没有服务器密码, 一直都是用的 authorized_keys, 还以为要歇菜了, 结果, 发现居然还可以在 Termius 里生成,

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1faphiq626aj21kw16o0x8.jpg)

然后, 是连到服务器,

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1faphipyjgrj21kw16oaeu.jpg)

**htop** 一个

![](http://ww1.sinaimg.cn/large/62fdd4d5gw1faphimzvhqj21kw0jugru.jpg)

想了想, 可以 **ssh** 到服务器, 那理论上应该也可以 **ssh** 到自己的 MacBook, 随手一搜, 还真有, **macOS** 自带就有功能, 也不用怎么折腾, 点几下就是.

[https://support.apple.com/kb/PH18726?locale=en_US](https://support.apple.com/kb/PH18726?locale=en_US)

但是, 发现, 好像只能内网?

搁置了一下想到, MacBook 如果连的 **VPN** 和 iPad 连的 **VPN** 是同一个服务器的话, 不其实也算在一个内网, 实验证明是可以的.

上图说话, MacBook 的 ~

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1faphit2safj21kw16on2x.jpg)

各种命令行跑起,

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1faphiu595tj21kw16o10t.jpg)

原生的键盘之上加多了一些开发常用到的键, **比如 ctrl, alt, tab, esc**

**Vim** 把代码写起来 = . =

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1faphitldyjj21kw16ogt7.jpg)

所以, 只带个 iPad 就上班是我想太多了咯?

---

### Dash

前不久, Mac 上著名的文档工具 Dash 开源了 iOS 版本, 不过没有上架到 app store

[https://github.com/Kapeli/Dash-iOS](https://github.com/Kapeli/Dash-iOS)

但是, **README** 上已经把整个安装流程描述的很详细了, 所以, 还是直接上图说话.

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1faphip4gzsj21kw16owks.jpg)

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1faphisfdc1j21kw16o455.jpg)

![](http://ww1.sinaimg.cn/large/62fdd4d5gw1faphipgxvsj21kw16ojy3.jpg)

---


### Touch Code Lite

代码编辑器(其实主要还是拿来看不是拿来写)

比如没事看看 Rails 的源码什么的

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1faphirj6chj21kw16oqfu.jpg)

不能 CtrlP 只好全局搜, 只能算凑合着.

![](http://ww2.sinaimg.cn/large/62fdd4d5gw1faphis2pg2j21kw16owng.jpg)

这货有点 Bug, 某些后缀的文件或者没有后缀的文件直接显示个未知 = . =

---


### iOctocat

Github 客户端

![](http://ww1.sinaimg.cn/large/62fdd4d5gw1faphini8lpj21kw16odjw.jpg)

![](http://ww2.sinaimg.cn/large/62fdd4d5gw1faphinv0unj21kw16o42j.jpg)

----


(◎_◎;) 取代 MacBook 暂时是没啥可能的, 但是, 如果每天有不少时间在路上各种交通的话, 可以试试这么玩.
