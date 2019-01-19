---
title: "ubuntu12.04 and Rails"
date: 2012-04-27T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

昨晚,把ubuntu升级到了12.04,其实,对我目前的使用来说,系统是否升级几乎没影响.不过看着更新管理器冒出了新版本,反正也就按几个下的功夫,于是就升上去了.

新版本多了首先感觉感觉性能上貌似快了些许,不多...另外对我而言直观的体验是launcher,之前的launcher也太敏感了,不过现在这倒是太呆板了,有时想要却不出来.不过桌面状态下不显示这点我比较喜欢.(以前也可以设置?不知道,木有研究)

另外alt键貌似上边会冒出个可输入命令行的,目前还木有发现其实际作用.

好吧,其实,我对这些改进啊有的没的还不怎么敏感.所以与其要看我介绍新功能新特性还不如直接搜索.

我敏感的只是,我的一些Rails项目都跑不动了.passenger报错

```ruby
Error message:
libmysqlclient_r.so.16: cannot open shared object file: No such file or directory - /home/xguox/.rvm/gems/Ruby-1.9.2-p290@Rails/gems/mysql2-0.3.11/lib/mysql2/mysql2.so
```

我还以为是我缺少了libmysqlclient_r.so.16这个文件,自己下载了之后也还是不行.期间尝试重装mysql,也试过之前说的那个,安装各种组建的命令还是无果.最终在Ruby-china得到大神指点,居然只需卸了重装mysql2这个gem就可以了.貌似是因为/usr/lib 下的mysql2.so没有link到gem里边的mysql2.so
过后,还是跟往常没有多大差别的继续Rails...