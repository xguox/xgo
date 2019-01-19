---
title: "My VIM Config"
date: 2012-04-14T16:01:23+08:00
draft: false
tags: []
categories: []
---

其实前不久就要写挺多东西的,只是忽然间事情就集中的涌了过来,状态没调整好,也就耽搁了些.之前是直接使用网上高人的vim配置以及插件,发现有很多自己也用不上,而且自己东添西改了之后就变得更臃肿了.于是,杠杠的重新整理.vim&.vimrc

首先是配色方案,可能是我的屏幕分辨率比较低的缘故,感觉好几个自带的配色方案挺不错的,但是相对却比较模糊.最后发现,fruity的那个配色比较好一些(主要是没有那种模糊感),不过配色方面不咋滴,于是自己在fruity.vim里头东改西改,最终个人感觉还不错.![](http://farm8.staticflickr.com/7136/6930980336_f2bc94c7c3_b.jpg)

and then就是一些插件了.目前主要是做Rails开发,以下几个插件是我较常使用到的: [NERD tree](http://www.vim.org/scripts/script.php?script_id=1658)、[snipMate](https://github.com/scrooloose/snipmate-snippets) 、[ctrlp](http://www.vim.org/scripts/script.php?script_id=3736)、 [ctags](http://www.vim.org/scripts/script.php?script_id=610)

NERD tree主要提供的是树型目录结构(如上图左),它的功能不仅这一点,丰富的多了,不过我感觉最强大最常使用的还是书签功能.简单的一个:Bookmark 命令,以后就不用一直在目录下跳转啊跳转

snipMate则是一个代码补全的插件,比如在controller下输入rp然后按下tab会直接输出`render :partial =>  "item"`在view里边输入rp然后按下tab的话则会直接输出`<%= render :partial => "file" %>`在view下使用都会智能的为你添加上<% %>或者<%= %>

补充一个,delimitMate,这个貌似也是一个补全代码的插件,不过我是作为snipMate的补充来用的,因为snipMate标点符号不会自动补充,比如输入 { 不会自动显示 }

很多人都说这类插件会阻碍学习熟记代码什么的,不过我是觉得,既然能方便、快速编写代码,何乐而不为呢,再说并不是所有的代码都自动补全(那就不用开发了),这个阻碍并没传说那么大吧.

ctrlp  个人比较依赖的一个插件,快速打开你想要的文件,这对于Rails这种MVC框架来讲是灰常有帮助的.印象中Sublime Text同样的功能也是ctrl+p这个快捷键组合.

ctags   在根目录下使用命令ctags -R后会创建一个tags文件,之后,你可以轻易地通过ctrl+]跳转到某个方法定义的地方.不过要提的一点是,一般的项目都能正常运行,但是我发现在公司的大项目下这个功能貌似有点问题,经常跳转不成功,不知道是否因为太多子目录的缘故.

另外说个Rails.vim,很多人都强烈推荐这个插件,不过装上后目前没发现什么功能是我特别需要的.

最后就是配置文件.默认字体,大小等等一些基本设置.

好吧,有兴趣的童邪直接往[我的github](https://github.com/xguox/MyVimConf)看过来吧,我把.vim挂上去了.
