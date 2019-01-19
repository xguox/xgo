---
title: "One Day With Linux"
date: 2011-12-31T16:01:23+08:00
draft: false
tags: []
categories: []
---

为了不影响其他人的网速,我只好选择了通宵下载VMware和Ubuntu.今早上老早就起来安装,看网上好多人安装Ubuntu各种各样的出错,而我却很幸运的一路顺风.不过中途有个"看电影"环节,估计单是那就花了半个钟,刚好那时候到了吃饭时间,于是就出去吃了个饭,回来基本搞掂.

Ubuntu给我的第一感觉就是界面很Nice.第二感觉是字体很Nice.只用过windows的我顿时觉得自己弱爆了,爆了,了...一直在windows都习惯了用chrome,Ubuntu自带的是Firefox,这俩都是网络上广受好评的浏览器.不过,我就是有那么一种惯性,用开了chrome就懒得再去试Firefox.不过既然是自带的话那我也不再去折腾装chrome了.我不是浏览器高手,基本快捷键都差不多就OK了,也不会不顺手什么的.PS:顺便吐槽下IE,连我的副标题都显示有问题.

我用Ubuntu无非就是体验在Linux下开发Ruby/Rails.于是基本配置搞掂之后就开始安装Ruby开发环境.我是用RVM来安装的,一开始的步骤都没什么问题,到了安装Rails的时候就出事了.
```
$ gem install Rails
```

当执行这行命令的时候,一直报错

```
ERROR: While executing gem ... (Gem::DependencyError)
Unable to resolve dependencies: Rails requires activesupport (= 3.1.3), actionpack (= 3.1.3), activerecord (= 3.1.3), activeresource (= 3.1.3), actionmailer (= 3.1.3), railties (= 3.1.3)
```

接着各种百度谷歌也没找着解决方法.到最后,我自己也忘记看哪里,然后突发奇想的就用了下面这行命令

```
$ rvm gemset install Rails -v=3.1.3
```

结果就很神奇的装上了.据说是GEM被GFW了的原因.不过淘宝最近推出了Rubygems 镜像.于是果断把Gemfile和Gem source改了.
然后是Git的配置,在SSH Key上除了点问题,不过跟着gthub的官方说明走也OK了.最近学到比较多的是Git的一些知识,比如就是,通过git clone获取的远端git库,只包含了远端git库的当前工作分支.其他分支都不见鸟.如果想获取其它分支,则要使用命令

```
git checkout -b 本地分支名 远程分支名
```
折腾了OS,VCS,最近我的Rails却停滞不前.还要受到最近期末考的一些干扰,心思难以投入Rails...悲剧.

昨天又划分了40G的空间给Ubuntu,原本硬盘就没剩几个G了.以后会更多的在Ubuntu下作业.比起windows,这里还多了一份宁静,可以让我更专注于开发与学习.
