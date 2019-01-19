---
title: "Linux 的 2>&1"
date: 2015-11-10T16:01:23+08:00
draft: false
tags: ["Linux"]
categories: ["Linux"]
---

Shell 命令缓慢的学习中...

Shell 脚本的 `>` 表示输出重定向, 而 `0 1 2` 则分别代表标准输入(stdin), 标准输出(stdout)和标准错误(stderr),

{% highlight shell %}
ls 2>1
{% endhighlight %}

会在当前目录执行 ls, 因为没有标准错误信息, 所以, 会产生一个空白的文件 `1`.

{% highlight shell %}
ls xguox 2>1
{% endhighlight %}

会生成一个名为 `1` 的文件, 里面的内容是
`ls: xguox: No such file or directory`

```ruby
ls xguox 2>&1

```
不会生成一个名为 `1` 的文件, 但是会把错误直接作为标准输出(即`1`)
`ls: xguox: No such file or directory`

所以, 也就可以解读 crontab 里面的命令了

```ruby
0 8-18/5  * * *  cd ~/apps/current && /bin/bash -l -c "RAILS_ENV=production bundle exec rake post:auto_post" 2>&1 >  log/init.log
```

最后,
在命令最后加 & , 是让命令在后台执行
