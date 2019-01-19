---
title: "Meteor: Understanding Publications and Subscriptions"
date: 2013-04-10T16:01:23+08:00
draft: false
tags: ["Translation", "JavaScript"]
categories: ["Translation", "JavaScript"]
---

##### LEVEL: INTERMEDIATE
Publications & subscriptions 是在Meteor里最基本也是最重要的概念之一, 但同时也是最难理解的. 很大程度上是由于这其中跟传统的构建web的方式有很大的不同.
过去, 我们习惯思考,定义API用在客户端和服务器之间进行数据传递, 而在Meteor中, 客户端和服务端的数据是同步的. 我们使用publications来精确地控制如何同步.
最初人们觉得这个概念有一些难以理解的部分原因是由于Meteor像"魔术"那样为我们做了这一切. 这些"魔术"都非常有用, 而具体发生了什么则都被掩盖封装起来了(就如魔术为何如此奇幻一样). 那么, 现在就让我们拨开这些魔术表层的面纱一探究竟. 在此, 我们将会学习到一到两个窍门.

## Defining Publications
从本质上讲, **publication**(使用**subscription**与之相连)是从服务端(源)collection到客户端(目标)collection的传递数据方法. 同时, 把subscription想象成一个漏斗连接着标准的数据存储(与mongodb数据库交互的源集合)与客户端缓存(目标集合, 相应数据的备份或者子集).

**subscription**精确地控制着哪些数据该通过这个漏斗, 同时负责同步两端的数据. 通过添加多个**subscription**到服务端数据存储, 我们就可以实时地, 有效地, 安全地保持各个客户端的数据同步.

这里头所用的隧道协议叫作 **DDP**(Distributed Data Protocol). 想要了解更多关于DDP可以观看Matt DeBergalis(Meteor的founder之一)在([The Realtime Conference](http://2012.realtimeconf.com/video/matt-debergalis)上的演讲. 又或者是 Chris Mather的[这个视频](http://www.eventedmind.com/posts/meteor-subscriptions-and-ddp)更详细的为你介绍DDP的概念.

现在, 我们了解了基础部分, 让我们往更深层探个究竟吧.

## Autopublish
当你创建了一个最基本的Meteor应用之后(比如使用 `meteor create`), 它会自动的启用 **autopublish**这个package. 首先的, 我们先来看看它究竟为干了些什么.

**autopublish**是移除了对subscriptions的需要还是只为你保管subscriptions 这取决于你如何看待它. **autopublish**所做的是自动地把服务端的所有数据镜像到客户端.
![](http://www.themeteorbook.com/images/book/autopublish@2x.png)

这是怎么做到的呢? 假设你在服务端有个`posts`的集合. 那么 **autopublish**会自动把在Mongo 的posts 集合所找到的每一个post发送到客户端给一个也同样叫posts 的集合.

所以, 如果你使用`autopublish`, 你就不用再去管subscriptions了. 数据在哪都可以访问, 所有事情变得各种简单. 当然, 很明显这是有问题的, 你不可能在每一个用户的机器上都缓存有你的整个app的数据库备份.

出于这个原因, **autopublish**只适合用在你的app刚起步还没考虑到subscriptions的时候.

## Publishing Full Collections
当你移除掉autopublish以后, 很快你就会发现你的数据都会在客户端消失不见. 有一种方法可以很简单地取回这些数据, 那就是简单的复刻`autopublish`, 并把一个collection全部publish. 例如:

```javascript
Meteor.publish('allPosts',function(){
  return Posts.find();
});
```
![](http://www.themeteorbook.com/images/book/fullcollection@2x.png)


同样地, 我们也还是把整个collection都publish了, 所不同的是, 现在对哪个collection进行publish是可控的. 在上边这个例子, 我们publish了`posts` collection, 而`comments` collection没有被publish.

## Publishing Partial Collections
更高一级的粒度控制是只publish某个collection的一部分. 比如, 只作用于属于某个author的posts:

```javascript
Meteor.publish('somePosts',function(){
  return Posts.find({'author':'Tom'});
});
```

![](http://www.themeteorbook.com/images/book/partialcollection@2x.png)

代码都很简单, 但是究竟在底层发生了些什么呢?

如果你有读过[Meteor的文档](http://docs.meteor.com/#publishandsubscribe), 可能会对`added()`和`ready()`在客户端设置记录的属性感到惊讶,  and struggled to square that with the Meteor apps that you've seen that never use those methods.(I am so sorry 这句没想着怎么翻译好)

这其中的原因是, Meteor提供了一个非常重要的便利 - `_publishCursor()`方法. 可能你从没见过它的使用? 因为有可能不是直接的调用,但如果你在调用一个publish函数并返回一个**游标(cursor)**(例如: `Posts.find({'author':'Tom'})`), 那么这就是`_publishCursor`.

当Meteor看到`somePosts`这个publication返回来一个游标(cursor), 则表明它自动调用了`_publishCursor()`publish这个cursor. 下面这是`_publishCursor()`所做的事情:

- 在服务端查找这个名字的collection
- 从cursor中取得所有匹配的documents并发送到客户端的同名集合(在这用到的是`added()`)
- 只要某个document被增删改, 都会被同步到客户端(在cursor上使用`.observe()`, 并使用`.added()` `.updated()` `.removed()`来完成)

那么,在上边的例子, 我们就可以很简便地确保只有用户感兴趣的posts(written by Tom)会出现在他们的客户端缓存之中.

## Publishing Partial Properties
在上边我们已经看到了如何publish我们的一部分posts, 但是我们还可以做的更精确一些. 下面看看如何publish特定的记录吧.
![](http://www.themeteorbook.com/images/book/partialproperties@2x.png)

跟前边的一样, 我们先用`find()`得到一个cursor, 不过这一次我们会排除掉一些

```javascript
Meteor.publish('allPosts',function(){
  return Posts.find({}, {fields: {
    author: false
  }});
});
```

当然的, 我们还能够结合这两种技巧. 比如我们想要返回所有来自Tom的posts同时, 可以这么写

```javascript
Meteor.publish('allPosts',function(){
  return Posts.find({'author':'Tom'}, {fields: {
    author: false
  }});
});
```

## 总结
现在, 我们已经知道了如何publish从所有collections的所有documents的所有属性(通过`autopublish`)到特定collections的特定documents的特定记录.

这包括了你能用Meteor的subscription所能做的所有基本东西, 而这些简单的技巧应该

有时候, 你可能会需要更深层的组合, 联接, 合并 publication, 而这些我们将会再找个时间详谈.

原文来自Tom Coleman [Understanding Publications and Subscriptions](http://www.themeteorbook.com/2013/04/05/publications-and-subscriptions/)