---
title: "Getting Started with Meteor"
date: 2013-03-05T16:01:23+08:00
draft: false
tags: ["Translation", "JavaScript"]
categories: ["Translation", "JavaScript"]
---

##### LEVEL: BEGINNER

当你看到这,也就说明你听说了[Meteor](http://Meteor.com/)这东西, 并对它产生一些好奇.
那就让我们创建一个最基本的应用来耍一耍吧. 希望能以此学习到一两样东西.

## 准备工作: 安装Meteor

要安装Meteor那就再简单不过了. 只需要执行下面这条命令
`curl https://install.Meteor.com | /bin/sh`

只要你是在[支持的平台](https://github.com/Meteor/Meteor/wiki/Supported-Platforms)上运行,跟着步骤来就基本没什么问题.

接下来让我们创建一个简单的app来确定是否安装成功.
`Meteor create forum`
如果安装成功, 那执行下面的步骤就可以把这个app跑起来了

```ruby
cd forum
Meteor
```

如果这一切都顺利的话, 那么在浏览器打开`http://localhost:3000`, 就会看到:
![](http://theMeteorbook.com/images/posts/bare-Meteor-app.png)

## 设置我们的forum

这个使用Meteor生成的简单的应用程序并不能做些什么; 只是或多或少示范了如何呈现模板和响应事件.

而今, 我们只需要用浏览器的控制台(比如Chrome的Inspector)来捣鼓一些我们这个应用的数据. 因此,我们不必太过在意现有的用户界面. 一切从简. 在创建一个posts集合并在一个列表中一一展示前,我们先在 forum.html 这么干:

```html
<head>
  <title>Forum</title>
</head>

<body>
  {% raw %}
  {{> posts}}
  {% endraw %}
</body>

<template name="posts">
  <h1>Posts</h1>
  <ul>
	{% raw %}  \{\{#each posts}}{% endraw %}
    <li>{% raw %}{{name}}{% endraw %}</li>
    {% raw %}\{\{/each}}{% endraw %}
  </ul>
</template>
```

当编辑完成上面这段代码后, 你应该会看到浏览器自动重载了页面. 现在, 除了一个"Posts"标题,你应该什么也看不到. 为什么会这样呢?

![](http://theMeteorbook.com/images/posts/empty-posts-list.png)

我们刚刚使用了`{%raw%}{{ #each }}{%endraw%}`这个helper来告诉Meteor创建了一个无序列表并且在其里边的每个li标签展示一条post. 但是在这里我们的posts为空. 所以页面中什么也没显示出来. 那么接下来让我们来生成一些posts吧.

在`forum.js`, 我们创建了一个叫`posts`的`Collection`, 并使用之前所建立的`posts`模板加载之:

```javascript
var Posts = new Meteor.Collection('posts');

if (Meteor.isClient) {
  Template.posts.helpers({
    posts: function() {
      return Posts.find();
    }
  })
}
```

这看起来只是一个简单的变化,但事实并非如此. 我们做的事是在客户端和服务端创建了一个叫作`posts`的Meteor `Collection`. 这是什么意思呢?

在服务端,当数据被添加进来之后, 会被插入写进到一个[mongo](http://www.mongodb.org/)数据库(mongo内置包含在每个Meteor应用程序). 当执行查询的时候, Meteor会查找到这个数据库. 这样我们其实就已经把这些posts永久性的存贮起来了.

在客户端,`posts` 集合会自动连接到服务端并在浏览器,本地镜像或者缓存中创建这个服务端集合. 这样, 当在浏览器中新建posts,这些posts就会直达服务端从而插入到mongo数据库中,并传输到其他连接上的本地的`posts`集合. 很难理解是吧?你很快就会知道这一切是怎么回事.

添加完以上的js以后, 你不会看到我们的应用程序发生任何的改变; 这是因为我们还没添加任何一条的数据. 那么现在开始添加吧.

## 在你的浏览器控制台中操纵数据


让我们来仔细看看这个神奇的集合. 在这我们会使用到浏览器控制台, 尽管我们假定你一直跟着我们的步伐并使用了Chrome的Inspector, 但其他的一些浏览器控制台(如Firebug等)也一样可以完成.

首先, 让我们看看集合里有些什么东西. 运行下面的代码,我们会告诉`posts`模板查找该集合中的数据:

```ruby
Posts.find()
» ‣LocalCollection.Cursor
```

`Cursor`是一个匹配指定的查询(在本例中为空)所得到的Meteor数据源. 现在看起来不那么有趣, 让我们使用下面这条语句来提取数据:

```ruby
Posts.find().fetch()
» []
```

由于还没有一条post, 这会返回一个空数组. 让我们添加上一条吧!

```ruby
Posts.insert({name: 'A Brand New Post'});
```

由于Meteor的响应式渲染( reactive rendering )使得HTML与底层的数据模型保持同步, 因此我们可以马上在页面上看到这条post
![](http://theMeteorbook.com/images/posts/single-post-inserted.png)

等一等,现在我们能够在客户端中从这个集合里抓取到这条post

```ruby
Posts.find().fetch()
» [‣Object]
```

我们能够在本地集合中看到这篇post的存在. 对此没什么概念吗? 试试刷新页面;或者在一个新的tab打开`http://localhost:3000`,这篇post还在是吧!

为什么会这样呢?在程序内部,Meteor已经同步了这个post对象(`{name: 'A Brand New Post'}`)到服务端, 并将其存储到mongo数据库中,这样,在所有的客户端都能访问到这篇post.我们同样能够在mongo的控制台中查看到它;在命令行中运行`Meteor mongo`(该命令运行的同时必须把项目跑起来),你也可以使用相似的接口来查看到:

```
> db.posts.find()
{ "name" : "A Brand New Post", "_id" : "a72d6722-262b-4dc5-80f6-564796a6cc95" }
```

花上一点时间想想这对你来说有多么的神奇 (with no work on your part!), 然后肆意地在浏览器的控制台操纵`insert`  `update`  `remove` 命令([文档在这](http://docs.Meteor.com/#collections)),同时在其他tab,其他浏览器甚至其他机器上(只要能看得到)看看同步的效果

原文转自 Tom Coleman [Getting Started with Meteor](http://theMeteorbook.com/2013/01/30/getting-started-with-Meteor/)








