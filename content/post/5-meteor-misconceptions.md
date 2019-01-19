---
title: "5 Meteor Misconceptions"
date: 2013-03-14T16:01:23+08:00
draft: false
tags: ["Translation", "JavaScript"]
categories: ["Translation", "JavaScript"]
---

由于某些原因,关于Meteor一直都有着各种各样的误解.或许是因为Meteor刚正式推出的时候, 缺失很多重要的功能.而很多不知道的是,这些缺陷在后来早就被修复了.

为了给Meteor正名,下面列举的是5个最常见的误解.


##### 误解#1:"Meteor的数据不安全"
在Meteor的第一个Demo推出之时, 数据安全和用户权限这些东西是没有的.任何客户端的用户都能够访问,修改数据库的数据然后发送返回到服务端.

但这第一个版本只是作为一个提示告诉大家Meteor来了, 并不意味着它可以被用来打造一个真实的应用产品. 打那之后, 现在的Meteor已经有了一个内置的身份认证包(authentication package).

**造成误解的另一个原因是, 许多新手为了更容易地快速上手常常在创建Meteor应用的时候, 遵循默认的把所有的数据发布给客户端.**

正如Meteor的开发者之一 **Avital Oliver**在[StackOverflow上的回答](http://stackoverflow.com/questions/10099843/how-secure-is-Meteor)那样

>当你使用Meteor的命令行工具创建一个应用的时候, 默认的会包含有两个packages:**AUTOPUBLISH**和**INSECURE**. 这两个packages能使得每个客户端都拥有所有的服务器数据库读写权限效果.这是非常好的原型工具(目的只为了开发),但通常这不适合用于应用的生产环境.当你需要发布到生产环境的时候,请去掉这两个packages

类似地,也因为Meteor可以共享服务端和客户端的代码,但实际上,这并不意味着所有的代码都可用于客户端.

所有放在`server/`目录下的东西都是只能在[服务端](http://docs.Meteor.com/#structuringyourapp)执行使用的, 同时Meteor也让你可以定义[服务端安全方法](http://docs.Meteor.com/#methods_header),这些方法可以在客户端调用.


##### 误解#2:"Meteor对SEO不友好"
与传统的网页应用不一样,Meteor不会在服务端搭建HTML页面.取而代之, 服务端只发送数据, 然后在客户端决定如何渲染之(这就是Meteor的"data on the wire"原则)

这种做法的缺点就是, 当JavaScript被禁用的话, 你的网站几乎跟一个空白页面无异.

你或许会觉得这会迷惑搜索引擎.但幸运地是, Meteor有解决的方案,那就是使用[Spiderable](http://docs.Meteor.com/#spiderable) Package.

![](http://theMeteorbook.com/images/phantomjs.png)

这个package使用的是[PhantomJS ](http://phantomjs.org/)在服务端预渲染后提供给网络爬虫.实际上网络爬虫得到的网页跟你看到的是一样的.


##### 误解#3:"Meteor不支持第三方packages"
有很多的讨论是关于Meteor所用的不是Node的标准包管理--NPM. 而事实上, Meteor压根就没有不允许使用第三方程序包.

And it's true that the vanilla install of Meteor does not come with a package manager. 但庆幸的是, [本书](http://theMeteorbook.com/)的合著者之一Tom Coleman, 同时也是Meteor包管理器[Meteorite](https://github.com/oortcloud/Meteorite)和packages资源库[Atmosphere](http://atmosphere.Meteor.com/)的开发者.

尽管当下Meteorite 和 Atmosphere还不是Meteor的官方部分, 但它们已经得到广泛的使用并将会在未来被合并到Meteor的核心部分.事实上,这方面的工作已经在Meteor的核心代码的[engine](https://github.com/Meteor/Meteor/tree/engine)分支中开展,预期会被很快解决.


##### 误解#4:"Meteor是完全封闭的(walled garden)"
一个密切相关的异议是,Meteor可以支持第三方packages,但这些第三方packages却严格地限制于流星的生态系统.

为什么有成千上万的Node packages 在那还要使用自定义的package呢?这又是否与开源精神相悖呢?

首先,需要意识到一点很重要的就是, 在Meteor中可以任意的使用node packages.尽管在当下这么做可能还会有一些棱角使得接口不那么顺利,但这种情况在`engine`分支添加[Node support](https://groups.google.com/forum/?fromgroups=#!topic/Meteor-talk/b6zQrgk8lYo)
并发布后将得到很大改善.

![](http://theMeteorbook.com/images/npm.png)
其次,Meteor不仅仅是Node.js的一个框架, 更是一种全新的网站应用开发方式和构想.

换句话说,想知道为什么Meteor不使用NPM无异于问为什么它不能使用Ruby Gems. 有很多人建议所有的JavaScript代码应该都用NPM来管理, 但这么做的话会有一些细节上的问题, 正如核心开发者[解释说](https://github.com/Meteor/Meteor/pull/516#issuecomment-12919473).


##### 误解#5:"Meteor只适用于原型"
"Meteor非常适用于小规模快速原型的项目,但它不适合用于大规模的应用."

在某种程度上, 这很难争个谁对谁错.毕竟Meteor的主页的版本号还是以`0`开头.

但事实是Meteor还没准备到一个黄金时间,比起内在的局限性,在如此青涩的时期(不到一岁)Meteor还有许多需要做的.

再说, 即使是现在, 有时牺牲一点稳定性来做交换得到Meteor带来的轻松且快速的开发还是值得的.如果你希望看到一些具体的Meteor案例,我可以推荐看看[Telescope](http://telesc.pe/)吗?

So,我坚信,在Meteor这样快速发展之下,今天的技术原型将会是明天thousand-user 的应用.


##### 总结
现在,即使我们正在写这本关于Meteor的书, 但我们首先依旧会承认一点, 如同其他的技术那样,Meteor同样也有着自己的缺陷.

But I do believe Meteor has enough potential to make it worth evaluating it on its own merits. 所以如果你犹豫不决是否使用Meteor, 希望我的建议能说服你你选择它.

原文来自 Sacha Greif  [5 Meteor Misconceptions](http://theMeteorbook.com/2013/03/12/5-Meteor-misconceptions/)