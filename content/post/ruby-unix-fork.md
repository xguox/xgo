---
title: "理解 Unix 进程"
date: 2015-04-28T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

推荐 Rubyist 看 <理解Unix进程>,

至少, 像我这种不是科班出身的, 可以好好补补一些计算机系统的那些基础知识.

虽然还没看完, 虽然书中的例子都是用 Ruby 去写的, 有一些章节理解起来还是有些费力.

前些天看到个好玩的, 在 irb 里面输入

```ruby
  if fork
    puts "entered the if block"
  else
    puts "entered the else block"
  end
```

输出结果:
![](http://ww2.sinaimg.cn/mw690/62fdd4d5gw1es7klghsxdj20jy0a4jsg.jpg)

if 和 else 都执行了.

通过

`ps -ef | grep irb`

可以看到有两个 irb 进程.

更直接一点, 退出 irb 重新执行:

```ruby
puts "parent process pid is #{Process.pid}"

if fork
  puts "entered the if block from ##{Process.pid}"
else
  puts "entered the else block from ##{Process.pid}"
end
```

![](http://ww1.sinaimg.cn/mw690/62fdd4d5gw1es7klik5t8j20sw0cumz2.jpg)

> if 语句块中的代码由父进程执行的, 而 else 语句块中的代码是子进程执行的. 子进程执行完后退出, 父进程则继续执行.

不过, 这里就纳闷了, 子进程没有退出吧, ps 还是出来两个 irb, 而且, 如果想继续输入代码是得隔着一个按键敲的, 比如要退出的话是 敲 eexxiitt.

= .  =

Anyway, 说到底其实是 Ruby 里的 fork 用法.
