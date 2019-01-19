---
title: "自动更换 Sublime Text 主题 by Automator IFTTT Dropbox"
date: 2013-12-24T16:01:23+08:00
draft: false
tags: ["Tools"]
categories: ["Tools"]
---

没事喜欢折腾下 Sublime Text 的主题, 当然可能看习惯一种主题对编码也有帮助的,  对各种语法匹配各种颜色形成了习惯, 出现 typo 或者其他的一些错误也较容易发现吧.

不过只是想在白天和黑夜交替更换一下 solarized 的 light 和 dark  , 大夜晚的用浅底的配色不幸福啊

1. 在[ IFTTT ](https://ifttt.com/)上创建一个 Recipe , Sublime Forum 里边的哪位大神用的是关联天气的日出和日落(if Weather), 好吧,  我还是直接设定时间好了(if Date & Time),  and  than   设置更换主题的时间,   IFTTT 居然不给同一个 Date & Time 设定两个时间 = 。 = 所以, 这里要用到两个 recipes(sunrise & sunset) ,  先做一个 [sunrise](https://ifttt.com/recipes/135801) 的.  时间设置早上9点(可能或有几分钟的触发延迟)

2. then **Dropbox**
-> create a  text file -> **Dropbox folder path** 不改了直接用 `IFTTT/Daylight`
-> **File name** 也是, 就 `sunrise`
-> **content**  则是 `light`

3.  **Automator**
新建一个**文件夹操作**,  接收文件夹选择 `/Users/xguox/Dropbox/IFTTT/Daylight`
-> 文件和文件夹 的 查找 Finder 项目 , 查找 `/Users/xguox/Dropbox/IFTTT/Daylight` 是否有文件拓展名为 txt 的.  -> 添加 **运行 Shell 脚本**

4.  在 textarea 中添加这段 shell script , 存储-搞定!


```python
DAYLIGHTDIR="/Users/xguox/Dropbox/IFTTT/Daylight"
SUNSETFILE="sunset.txt"
SUNRISEFILE="sunrise.txt"

if [ -f "$DAYLIGHTDIR"/"$SUNRISEFILE" ] || [[ -f "$DAYLIGHTDIR"/"$SUNSETFILE" ]] ;
then
   CURRENTSETTING=`grep tmTheme ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User/Preferences.sublime-settings | awk -F'[.|.]' '{print $2}'`
   DAYLIGHTFILE=`ls "$DAYLIGHTDIR"`
   NEXTSETTING=`cat "$DAYLIGHTDIR"/"$DAYLIGHTFILE"`
   sed -i '.bak' 's/'$CURRENTSETTING'.tmTheme/'$NEXTSETTING'.tmTheme/' '/Users/xguox/Library/Application Support/Sublime Text 3/Packages/User/Preferences.sublime-settings'
   rm -f "$DAYLIGHTDIR"/"$DAYLIGHTFILE"
fi
```

重复一次建一个 sunset 的recipe, 调好时间把 **content** 的 light 改为 dark 就是了.

关于 Shell 好吧, 尴尬我没认真学习过, 只能大概看懂然后做些修改( xguox 的配色来自 [Base16](http://chriskempson.github.io/base16/#default) 不是自带的 ).  其实我是在终端先把脚本测试运行了一通滴!比如 awk 那里就没看懂, 直接运行了那一行才把原本的`[(|)]`改成了`[.|.]`, 理论上其实可以无限更换, 只是太花哨频繁了也不甚好.



IFTTT (Dropbox)+ Automator(Shell) 这个组合赞死了! Google 之发现很早就有人这么干了, 看来还是 out 了.

参考自[Sublime Forum](http://www.sublimetext.com/forum/viewtopic.php?f=4&t=13662)

题外话扯淡,  本来说试试用简书的富文本, 发现貌似从原本的 wysihtml5 换成了 redactor. 靠!林总果然是土豪啊! 一个编辑器也要花上 $199 . 不过单看这一般功能看不出比一些开源的货有啥流弊之处。