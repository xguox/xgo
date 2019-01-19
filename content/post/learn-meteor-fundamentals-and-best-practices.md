---
title: "Meteor 基础与最佳实践"
date: 2013-03-08T16:01:23+08:00
draft: false
tags: ["Translation", "JavaScript"]
categories: ["Translation", "JavaScript"]
---

##### 引言

想知道这个新奇又好玩的东西--
Meteor是如何工作的?那就太好了,你来对地方了. 我会向你展示一个Meteor项目的方方面面 并会给你一些最好的提示让你应用在未来自己的Meteor项目当中.

##### What is Meteor

Meteor能让你打造非常动态的的网页同时代码量出乎意料之少. 记住一点,Meteor目前还在一个超级测试的版本(原文发布时候为 preview 0.3.8, 翻译时为 preview 0.5.7). 所以, 请不要因为工作的不够完美而感到意外.
Meteor是基于[Node.js](http://nodejs.org/), 因此, 你所写的大部分代码也将会是JavaScript. 这没什么好惊奇的. 如果你想要一些好的资源来快递提高你的JavaScript, 可以看一看 [JavaScript Garden](http://bonsaiden.github.com/JavaScript-Garden/).
Meteor存储数据所使用的是MongoDB. 尽管如此, 实际上,Meteor使用的是minimongo作为接口.  它只是拥有相对完整的功能而非标准MongoDB的所有功能. 你并不需要确切地知道MongoDB是如何工作的, 但我还是建议你看一看Meteor 的[Collections文档](http://docs.Meteor.com/#collections)以便知道你能做些什么.
Meteor目前使用的模板引擎只能是 [handlebars](http://handlebarsjs.com/). 但在未来支持使用别的模板引擎也是很有可能的.(翻译该文章时已经支持使用其他的如 Jade )
最后, 只要你开发网站, 了解 HTML 和 CSS 都是必须的.

##### 基础

一个Meteor项目包含的大多都是JavaScript文件. 如果你放置任意一个 `*.js` 文件在你的项目的任何位置, Meteor会自动帮你加载并运行之. 你在Meteor项目中写的每一个JavaScript文件都会被部署到服务端和客户端(额...也不完全是这样的,继续往下看吧!). 之所以Meteor这么,这么的cool,原因之一就是:当你写了一个JavaScript函数,在客户端和服务端都可以使用之!
更cool的是, 比如,你在项目的任意一个地方放置 `*.less`, 那么这些文件都会自动被编译并发送至客户端以及从而包含进页面当中.
有时候,可能你希望分离开服务端和客户端的代码. 幸运地是,

Meteor有这么一对标识可以帮你: `Meteor.is_server` 和 `Meteor.is_client`
下面的例子在浏览器的JavaScript控制台中会输出 `Hi. I'm CLIENT`,而在服务端中则会输出 `Hi. I'm SERVER`

```javascript
// This function is available on both the client and the server.
var greet = function(name) {
    console.log("Hi. I'm " + name);
}

// Everything in here is only run on the server.
if(Meteor.is_server) {
    greet("SERVER");
}

// Everything in here is only run on the client.
if(Meteor.is_client) {
    greet("CLIENT");
}
```
这非常的简单. 在客户端和服务端共用代码使你能够做到最大化的代码复用,这大大地减少了开发时间.

##### 项目文档结构

可能, 你无数次地希望不再服务端和客户端之间共享所有的一切. 比如你有一些私密的算法在服务端执行,而你绝不希望Meteor把这些算法发送至客户端让全世界都看得见. Meteor提供了两个"特殊"的目录来帮助分离客户端和服务端的代码:```[project_root]/client/```和```[project_root]/server/```. 在server目录的JavaScript只会在服务端执行而不会发送至客户端, 反之亦然. 这就使得我们无须在各个地方都使用```Meteor.is_client```  和 ``` Meteor.is_server```.取而代之, 对应的放置你的代码就OK了.

项目的文件结构非常重要, 尤其在考虑到文件的先后加载顺序的时候. 假定有两个文件, 其中一个依赖于另外一个, 你知道你的JavaScript的加载顺序吗?  其规则如下:

1.   `[project_root]/lib`里的文件会最先被加载. 显而易见的,  库需要放在这个文件
2.  文件会根据目录的深度来进行加载排序,放在深一层的文件会先加载
3.  同一级的文件会按照字母顺序来排序加载.
4.   ```main.*``` 会最后加载, 当有代码需要在其他所有的库或脚本之后执行这会很有用.

Meteor有一些特殊的目录来帮助你解决分离 `client/server`的代码以及加载的顺序:

- `[project_root]/lib/`  该目录下的代码会在你的 client/server 代码执行之前加载
- `[project_root]/client/` 这个目录下的代码只会在浏览器端而不会在服务端执行
- `[project_root]/server/` 这个目录下的代码只会在服务端而不会客户端执行
- `[project_root]/public/` 静态文件放在这个目录下, 并且你可以很轻松地在你的html中指向 image.jpg
- `[project_root]/.Meteor/` 一些特殊的,和项目相关的的信息放在这里,  比如你安装的某些模块. 你几乎可以不用关心这里边的东西

##### 响应(Reactivity)

Meteor 通过 响应数据源("reactive" data sources) 和 上下文(context) 帮你省去了当数据改变时手动替换页面的麻烦. 当响应数据源被更新后,响应上下文会重新运行(A reactive context is just a function that will get re-run if it contains a reactive data source that gets updated.) 刚开始的时候要把思维转换过来可能有些难, 下面的例子将为你解释清楚.
下面是一个html页面和一个叫`cool_dude`Meteor[模板](http://docs.Meteor.com/#templates), 以及一个在客户端运行的JavaScript函数--
传一个`username`的值给模板用以渲染.

```html
<html>
  <head>
  </head>
  <body>
    \{\{> cool_dude }}
  </body>
</html>
```

```html
<template name="cool_dude">
  <p class="important">{{ username }} sure is one cool dude!</p>
</template>
```

```javascript
// On the client:
Template.cool_dude.username = function() {
    return "Andrew Scala";
};
```
当页面被渲染的时候会输出 `"Andrew Scala sure is one cool dude!" `

模板即是响应的上下文: 如果它依赖于响应数据源, 那么当数据源改变时它会重渲染自身. 客户端的`session`一种响应数据源. 它只会在客户端存储一些键值对, 且当页面刷新的时候被擦除.

让我们把上面的例子模板上下文改为响应数据源:

```javascript
// When the app starts,
// associate the key "username" with the string "Andrew Scala"
Meteor.startup(function() {
    Session.set("username", "Andrew Scala");
});

Template.cool_dude.name = function() {
    return Session.get("username");
};
```

现在,模板将会在`Session`里取到`username`的值. 在响应上下文我们有一个响应数据源. 如果存储在`Session`的`username`的值更改了, 模板会重新渲染新的值到页面上. 如给`username`设置一个新的值:

```javascript
Session.set("username", "Bill Murray");
```

只要执行了这行代码,不管在哪, 页面都会立马输出 `Bill Murray sure is one cool dude!`
在这里, 我会列出其他的响应上下文和数据源,你也可以到 Meteor的[Reactivity文档](http://docs.Meteor.com/#reactivity)自行阅读.

##### 发布/订阅

**Note: 在每一个项目的根目录下执行 `$ Meteor remove autopublish`, 否则它会默认发布你的所有数据到客户端, 这显然不是个好的做法**

服务端发布数据给客户端使用, 同时,客户端向服务端订阅已发布的数据. 在刚开始, 想要明白服务端发布数据和客户端订阅数据的关系会有些困难. 常规的经验是这样的: **客户端只能访问当前时间点需要操作的数据, 除此之外的均不能访问**. 举个例子, 如果你有一个聊天应用, 客户端不能够接收你的网站上每一个频道的所有信息, 而只能是当前所访问的频道的信息. 同样的也不能够知道别的频道的用户.

下面这是一个关于 发布/订阅 的方面教材,  客户端可以看到数据库中的每一条信息:

```javascript
var Messages = new Meteor.Collection("messages");

if(Meteor.is_server) {
    Meteor.publish("messages", function() {
        return Messages.find({});
    });
}

if(Meteor.is_client) {
    Meteor.subscribe("messages");
}
```

当在客户端`Messages.find({})`,可以得到数据库里的每一条信息.
我们可以指定一个参数使得在订阅的时候缩小范围只获取实际需要的信息(如在`"cool_people_channel"`频道中的信息)

```javascript
var Messages = new Meteor.Collection("messages");

if(Meteor.is_server) {
    Meteor.publish("messages", function(channel_name) {
        return Messages.find({channel: channel_name});
    });
}

if(Meteor.is_client) {
    Session.set("current_channel", "cool_people_channel");

    Meteor.autosubscribe(function() {
        Meteor.subscribe("messages", Session.get("current_channel"));
    });
}
```

[Meteor.autorun](http://docs.meteor.com/#deps_autorun)是一个响应上下文, 意思是只要有响应数据源更新,所有在里头的东西都会重运行. 我们把当前所在的频道存储在Session的`"current_channel"`. 如果这个session值改变, 那么订阅(subscription)也就更新,
这样我们就能够访问其他的信息了. 如果用户想要加入`"breakfast talk"`这个频道. 我们可以运行`Session.set("current_channel", "breakfast_talk")`, 这会触发autosubscribe, 让我们可以且仅可访问`"breakfast_talk"`频道下的信息.

或许,你很多次都希望发布所有的collection到客户端. 仔细思考客户端实际需要的是什么. 比起发送所有的文档, 只发送特定领域会来得明智些.

##### 服务端方法

由于客户端不允许查看数据库中有什么东西, 你一对会好奇客户端又是如何存储信息的. 解决办法是使用Meteor的[服务端方法(Server Methods)](http://docs.Meteor.com/#methods_header). 你在服务端定义了所有的函数用以一些危险的做法比如修改和更新数据, 然后让客户端像调用常规方法那样call这些函数并返回值. 客户端永远也不会看到具体的实现,也不会自己修改数据. 这些都是服务端做的事.
想要添加一个用户到数据库中, 假定有一个叫`create_user` 的方法--传入一个username并让服务端插入一条记录. 它会返回给客户端一个`ObjectID`这样客户端可以抓取到这个用户的信息并做任何接下来要做的事.

```javascript
if(Meteor.is_server) {
    Meteor.methods({
        create_user: function(username) {
            console.log("CREATING USER");
            var USER_id = Users.insert({name: username});
            return user_id;
        },
    });
}

// Remember, the client's browser only ever sees the code below:
if(Meteor.is_client) {
    var username = "Andrew Scala";

    Meteor.call("create_user", username, function(error, user_id) {
        Session.set("user_id", user_id);
    });
}
```
在这个例子中我的 user_id被设置到客户端session中, 当任意模板使用到我的user_id
 的时候均立马可以得到更新.

##### 保护你的数据

默认的, 你可以在客户端打开一个JavaScript 控制台并**运行数据库查询**.这非常的不安全.最糟糕的情况是, 当我们在访问你这么cool的Meteor应用时可以运行`Users.remove({})`来擦除你的所有数据.

Meteor即将会做一些措施来更好的保护你的数据, 但截至目前为止这是唯一的方法可以做到保护的. 这些代码被包含在了基于 Meteor的 [madewith](http://madewith.Meteor.com/)网站中.这个代码片段阻止了从客户端进行insert/update/remove等操作. 把下面这段代码放在你的服务端代码中的任意位置:

```javascript
// Relies on underscore.js. In your project directory:
// $ Meteor add underscore
Meteor.startup(function() {
    var collections = ['collection_name_1', 'collection_name_2'];

    _.each(collections, function(collection) {
        _.each(['insert', 'update', 'remove'], function(method) {
            Meteor.default_server.method_handlers['/' + collection + '/' + method] = function() {};
        });
    });
});
```
##### 敬请关注

准备好开发一个真实的Meteor项目了吗?一起期待吧, Part 2 就在路上了.届时将引导你完成一个完整的app!

原文来自Andrew Scala [Learn Meteor Fundamentals and Best Practices](http://andrewscala.com/meteor/)
Ps:文章有点老,XD...不过就像该篇文章的作者所说, Meteor 现在还是处于一个 super-beta 阶段, 版本更新自然会相对来得快些. 据说1.0版本将会在不到一年的时间内发布.




