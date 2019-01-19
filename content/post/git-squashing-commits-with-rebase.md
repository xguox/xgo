---
title: "Git 合并 commits"
date: 2015-12-22T16:01:23+08:00
draft: false
tags: ["Tools"]
categories: ["Tools"]
---

很多时候, 为了方便别人帮忙 review 代码, 习惯喜欢把某个 todo 任务的提交合并作为一个.

如果想直接合到前一个 commit 的话, 一般都是用

**`git commit --amend`**

很多时候 typo 都这么干, 省的多一个 commit.

而当不是简单的 typo, 并且这个 todo 分支在开发过程中提交了比较多的 commit 时候, 用到的方法有两个, 这种情况还是比较多的, 尤其某个 todo 较为复杂.(当然, 使用的前提是这些分支提交的 author 都是你自己)

第一种:
极其简单粗暴, **直接 `git reset` 到过去的某个 SHA, 再重新 commit.**
= . =
目前没发现什么副作用, 因为 reset 只是把 commit 变为 unstage 状态, 并不会直接删除你在这些提交中的更改, 好吧, 如果你想  discard changes 的话再来一发 `git checkout`, 那就白干了 (￣(エ)￣)ゞ

第二种:
**使用的是 `git rebase`**
这里借 ruby-china 的项目来说明吧.
![](http://ww2.sinaimg.cn/large/62fdd4d5jw1f236vx2dopj21qq06g786.jpg)
最近几次的提交是这样的, 假设最后的三个提交都是某个 todo 任务的相关提交(好吧, 其实这里不是相关的, 只是假设), 如果要把它们合并为一个的话

`git rebase -i HEAD~3`

![](http://ww1.sinaimg.cn/large/62fdd4d5jw1f236vy3ae8j21380n4aen.jpg)

返回的结果是酱紫的, 然后把后两个的 pick 改为 s, 保存退出, 会显示提交信息的编辑,

![](http://ww4.sinaimg.cn/mw690/62fdd4d5jw1f236vyvp3yj20yu0rggqi.jpg)

自己在手工输入提交信息或者直接用默认的都可以. 除了 pick, s(quash)以外还有几个用法, 不过用的比较少 = . =

最终结果, 可见, 最后三个 commit 被合并为一个了.

![](http://ww2.sinaimg.cn/large/62fdd4d5jw1f236vzihj5j219803s40g.jpg)
