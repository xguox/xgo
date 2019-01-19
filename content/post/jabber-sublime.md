---
title: "扯点Sublime"
date: 2013-04-11T16:01:23+08:00
draft: false
tags: ["Tools"]
categories: ["Tools"]
---


#####  Fxxk Jabber


回想起初初接触编程语言的时候, 特喜欢折腾IDE的玩意, 什么visual studio啊, Eclipse啊, Netbeans, Aptana的会不会用好不好用都不管了, 装进PC再说, 怎么说, 很傻13的觉得我的PC里有这些工具我就牛13了, 熟不知真正的编程知识却木有学到, 后来搞Ruby开始放弃笨重的IDE改玩Vim和Sublime Text, 当时Textmate被捧上天不过木有Mac耍不起.
换了Mac了, Textmate2也开源了, 于是也装上耍了耍. 没多久又继续用Sublime Text了.  OOXX来来去去, 这两货其实真没差多少. 好吧, 是我用的都不够透彻. (T . T)


##### Theme

有时候蛋疼起来可以折腾某个主题某个配色好长的时间, 即使这对我的编程技能木有任何实质性提高.  强推一款theme - [Flatland](https://github.com/thinkpixellab/flatland),  随着Flat UI的兴起, ST的主题肿么能落后啊.  附带的icon更是比Sublime原本的霸气的不是一点两点啊. 貌似在哪见过, 想不起来了.

![](http://farm9.staticflickr.com/8392/8636887917_4db9f0f2b7_b.jpg)

这UI还行吧? 我自己也懒得再去调了, 额外的把cursor换了个色就是了(原本的灰色真心不够骚啊, 老是眼瞎找不着)

说到theme就想起这个正在用着的[Knockdown](https://github.com/aziz/knockdown) Package -Markdown文件专属的主题, 再配上Sublime Text的**Enter Distraction Free Mode** 用写静态Blog很带感叻, 对不对.  有时候, 巨爱这个模式. 舍不得Sublime的原因之一

![](http://farm9.staticflickr.com/8264/8638936015_31cde7d54f_b.jpg)

之前也试用过[MarkdownEditing](https://github.com/ttscoff/MarkdownEditing), MarkdownEditing的高亮做的没Knockdown那么好(说实话, Knockdown的代码块的高亮配色不怎么符合我的style, 太过偏暗偏沉, 但总比木有高亮来得强), 挺喜欢ME的默认放在center这一点, 但是, 在**Enter Distraction Free Mode**下, 这个优点也就荡然无存了.


##### Packages

每个用Sublime新手老手都会介绍强推的[Package Control](http://wbond.net/sublime_packages/package_control/installation) 插件管理就不再累赘啦.
Actually, 想装啥插件可以直接在里头搜, 或者网页版搜索, 一般都会给出相应的github开源链接. 装什么插件就更是因人而异的.  比如如果从来不写Markdown的那就木有必要装我上面说的那货.  又比如, 如果是重度Hacker News 用户的话, 直接搜hacker news(其实在"Install Package"上输入hack就已经跑出来一堆相关让你选择), 当然也不是啥都有的, 如果搜的是reddit就没看到结果了.

个人用的目前装着的packages有:

- [All Autocomplete](https://github.com/alienhard/SublimeAllAutocomplete) 这货是在已打开的文件的基础上拓展匹配的
- [knockdown](https://github.com/aziz/knockdown)
- [Sass](https://github.com/nathos/sass-textmate-bundle) Sass的高亮, 补全...
- [CoffeeScript](http://xavura.github.io/CoffeeScript-Sublime-Plugin/)  CoffeeScript高亮,补全, Compile等...
- [JSHint](https://github.com/uipoet/sublime-jshint) 检查Javascript的语法规范, 要先在装了node, 然后`npm install -g jshint`
- [Emmet](http://emmet.io/) 前身Zen Coding, 貌似所有的packages之中排在最前的. 几乎被吹捧成为神器中的神器了.
- [HTMLpretty](https://sublime.wbond.net/packages/HTML-CSS-JS%20Prettify) 个人觉得解决缩进等格式问题的神器, 好吧, 即使木有这个偶本身也是具有良好编码规范的.
- [Git](https://github.com/kemayo/sublime-text-2-git)
- [Web Inspector](https://github.com/sokolovstas/SublimeWebInspector) 纯属贪新鲜装上耍耍, 取代浏览器的inspector不太可能.
- [ERB Insert and Toggle Commands](https://github.com/eddorre/SublimeERB) 快速输入`<%= %>`或者`<% %>`
- [Hacker News](https://github.com/dotty/HackerNews-SublimeTextPlugin) 不常用, 而且要自己改改shortcut, 否则会占用了显隐侧边栏的快捷键.
- [PlainTasks](https://github.com/aziz/PlainTasks) TODO插件一枚. 配合Sublime的各种快捷键各种操控很nice


##### Settings-User

先把custom的show一下, 也没啥的, 都是些无关痛痒的修改. 个人爱好吧. 有个必须有的就是`"scroll_past_end": true,` 经常写了一大车页面布满了, 下面的就滚不起来, 眼睛老要往下瞄很不爽. 设置以后就可以自由控制底部显示.
基本上自己看着Settings - Default 还是老规矩按需修改啦.

```javascript
{
    "theme": "Flatland.sublime-theme",
    "color_scheme": "Packages/Theme - Flatland/Flatland.tmtheme",
    "tab_size": 2,
    "font_options":
    [
        "bold"
    ],
    "font_face": "Courier New",
    "font_size": 20.0,
    "bold_folder_labels": true,
    "hot_exit": false,
    "open_files_in_new_window": false,
    "create_window_at_startup": false,
    "scroll_past_end": true,
    "wide_caret": true
}
```


##### Key Bindings - User

快捷键同样跟Settings可以多看Default绝对比看路边3322的像我这些要靠谱, 完整的多. 忘了在哪学到的`{ "keys": ["ctrl+command+r"], "command": "reveal_in_side_bar" }` 挺实用的, 直接在侧边栏显示当前的文件所在的目录.

```javascript
[
  { "keys": ["ctrl+command+r"], "command": "reveal_in_side_bar" },
  { "keys": ["command+shift+."], "command": "erb" },
  { "keys": ["super+v"], "command": "paste_and_indent" },
  { "keys": ["super+shift+v"], "command": "paste" }
]
```


##### Shortcut

列一些个老是忘记用快捷键的操作, (有时候又无故发现一些快捷键)

```ruby
focus到侧边栏: 'ctrl + 0'
Command + D:选择词.重复按下可以增加选择下一相同的词.按下 Ctrl + Command + G 可选中所有相同的词.
Command + Option + /:块注释.
Control + M:跳转到对应的括号.
Control + Shift + M:选中当前括号内的内容,重复按下可增加选择括号本身.
Command + Shift + J:选中当前缩进级别下的所有代码.
Command + Option + .:闭合 HTML 标签.(说常用也不常用, 但又貌似有用)
按住option 拖拽多列
按下 Control + Shift + 方向键,可以选中矩形区域的文本.
选择数行文本末端,选中区域然后按下 Shift + Command + L.

```


##### PS:

官方的symlink貌似有点不妥,

```ruby
ln -s "/Applications/Sublime Text 2.app/Contents/SharedSupport/bin/subl" ~/bin/subl

```

稍作修改为:

```ruby
ln -s "/Applications/Sublime Text 2.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl

```