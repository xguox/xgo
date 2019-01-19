---
title: "Introduction to Realtime Web with Meteor and Node.js"
date: 2013-03-10T16:01:23+08:00
draft: false
tags: ["Translation", "JavaScript"]
categories: ["Translation", "JavaScript"]
---

最近,有许多关于 [Derby.js](http://derbyjs.com/)的激动人心的讨论涌现在了我的Twitter Timeline. 我从未使用过能够帮你做这么多--实时同步客户端和服务端 的框架. 从本质上讲, 这使得我们可以自己编写一个代码量很少的应用可以让两个人编写同一个 text field–live. 而 Derby帮你处理了在 models 和 views 之间所有的同步. 就如 Google Docs 的协作编辑那样.

这非常的伟大, 但经过深入的研究, 发现 Derby.js 并没有我想象中的那么成熟--目前还没有到1.0版本. 当然, Node.js 和 Meteor 也同样没有. 但相比起来, 似乎Derby缺少的更多一些. 比如, 据我目前知道的, 还没有一个好使的方法来处理sessions. 或许是因为缺乏文档的原因吧, 但是, 据说Derby的团队目前正在开发authentication. 如果有谁有一些关于Derby.js 处理sessions的新手指引, 我会很乐意去研究的.

另外一个我经常见到被拿来与Derby.js做比较的框架是[Meteor](http://Meteor.com/). 与Derby相似的是,它也能在多个客户端下实时更新views, 尽管做法上可能跟Derby有点不同. Derby可以较容易的使用多种数据库, 而Meteor则只亲近于[MongoDB](http://mongodb.org/). 事实上, 通过如[Mongoose](http://mongoosejs.com/)客户端接入数据库的API与你在服务端所期望的已经非常接近了.

比起现在node的一些缺点和[争议](http://bjouhier.wordpress.com/2012/03/11/fibers-and-threads-in-node-js-what-for/), Meteor看起来是非常有趣的选择用来建立有实时需求的应用. 个人觉得还是Derby基于传统回调的编程形式更吸引我, 但在Derby的强大背后,却缺乏健壮的文档和一个大的开发者社区, 这无疑是个很大的打击. 或许这会随着时间推移而有所改变吧, 但比起Meteor来说还是会慢很多, 因为后者最近获得了1100万美元的资金. 这笔财政资金确保了Meteor的存在以及得到持续的支持. 对于那些需要财政与发展稳定的框架的开发者而言, 这笔资金只会让Meteor更加优胜.

今天,让我们一起来看看如何新建一个真实的但又简单的Meteor应用. 本质上说, 这是基于[Tom的 Vimeo screencast](http://vimeo.com/40300075)的一个新手指引. 与Tom的 Vimeo screencast最大的不同是处理事件的方式. 比起复制粘贴一个Meteor示例的代码, 我会一步一步的通过自己的方式来处理使用Enter键来提交一则讯息. 让我们开始吧.

##### Creating a Meteor App
Derby和Meteor 他们共有的一个大加分是他们各自的命令行工具. 与Derby使用Node的内置的 npm 工具所不同的是, Meteor使用的是它自己的.

在终端(Mac OS X 和 Linux),执行如下的命令. (在这之前请确保你已经安装了Node)

```ruby
$ curl https://install.Meteor.com | /bin/sh
```

接下来的事Meteor会自己搞定了.

要新建一个项目, 先转到你的工作目录然后运行下边的代码. 这会创建一个目录, 里边包括有Meteor和一个最基本模板程序.

```ruby
$ Meteor create chat
```

现在, 你可以转到该目录并运行下面的代码让它跑起来

```ruby
$ cd chat
$ Meteor
Running on: http://localhost:3000/
```

想要看到这个最基础的应用程序, 你只需要在任意一款不过时的浏览器下打开`http://localhost:3000/`

只要你想, 你可以使用Meteor内置的`Meteor deploy`命令来部署你的应用到Meteor自己的服务器上

```ruby
$ Meteor deploy my-app-name.Meteor.com
```

只要你更新保存了你的代码, 所有连接上的浏览器都会实时更新其页面.

##### Developing the Chat App

通过`Meteor create`命令生成的文件夹里, 你可以看到几个不同的文件. 如果你的系统设置了显示隐藏文件, 那还可以看到一个`.Meteor`文件夹. 这个文件夹包括的Meteor本身, 以及MongoDB数据库的文件.

在你的项目的根目录, 你可以看到的还有`chat.html` `chat.js` `chat.css`. 这三个文件自解释?的. HTML包含了这个应用的模板和视图, 并都通过`chat.css`来添加样式. 而这个JavaScript文件则包含了在客户端和服务端执行的代码. 这非常的重要--请不要把任何东西比如配置数据和密码都放在这里, 因为任何人都能通过查看你的应用程序的源文件看到.

用你最喜欢的文本编辑器打开`chat.js`. 我个人使用的是[Sublime Text 2](http://sublimetext.com/), 因为它很简洁并且有多光标功能.
你可以在`chat.js`看到下面这些代码

```javascript
if (Meteor.is_client) {
  Template.hello.greeting = function () {
    return "Welcome to chat.";
  };

  Template.hello.events = {
    'click input' : function () {
      // template data, if any, is available in 'this'
      if (typeof console !== 'undefined')
        console.log("You pressed the button");
    }
  };
}

if (Meteor.is_server) {
  Meteor.startup(function () {
    // code to run on server at startup
  });
}
```

分别注意到`if`语句里的`Meteor.is_client`和`Meteor.is_server`部分. 在这里边的代码分别只在客户端和服务端下执行. 这就是Meteor共用代码的能力了.

删除掉`Meteor.is_server`这段代码并把`if (Meteor.is_client)`里边的代码删除了,只留如下部分:

```javascript
if (Meteor.is_client) {

}
```

注意到, 当你保存了你的脚本之后, 浏览器会马上重新加载这个新脚本.

##### Creating the View

在我们正式对这个脚本文件动工之前, 我们需要先新建一个视图用来展示聊天记录.
在编辑器里打开`chat.html`并删除`body`标签里边的代码. 包括名为hello的`template`标签.只留如下部分

```html
<head>
  <title>chat</title>
</head>

<body>

</body>
```

接着在`body`标签里添加下面这句

```html
\{\{ > entryfield}}
```

Meteor使用的模板系统与[Mustache](http://mustache.github.com/)很相似.大括号\{\{}}`表示要呈现的内容. 通过简单地在两对大括号里添加内容如{{hello}}`, 模板系统会用`hello`这个变量的值来替换它. 后面会更详细的介绍.

注意到了在`entryfield`这个词前面有个大于号`>`了吗? 使用该符号来指定渲染哪一个模板.

接下来使用下面这段代码来新建一个名叫`entryfield`的模板

```html
<template name="entryfield">
    <input type="text" id="name" placeholder="Name" /> <input type="text" id="message" placeholder="Your Message" />
</template>
```

在这个例子中,`template`标签只有一个属性--用来表示这个template的名字. 也就是当我们渲染的时候需要用来指定的名字.

查看浏览器, 你会发现页面已经刷新了, 输入框已经呈现出来了.

接下来, 在body里边添加另外的一个mutache标签用以渲染讯息列表

```html
\{\{> message}}
```

最后, 我们还需要新建一个名叫`messages`的模板. 在`entryfield`模板下面添加下面这段代码

```html
<template name="messages">
    <p>
       \{\{#each messages}}
            <strong>{{name}}</strong>- {{message}}
       \{\{/each}}
	</p>
</template>

```
注意到`each`子句. 在Meteor中你可以使用如下的语法来遍历一个数组模板

```html
\{\{#each [name of array]}}
\{\{/each}}
```

使用each循环时,上下文会有所改变. 当引用变量的时候, 实际上你引用的是每一个数组元素的值.
例如,在我们的chat应用中, 我们遍历了数组模板"messages"里边的每个元素, 该数组可以像下面这样,

```javascript
[
    {
        "name": "Andrew",
        "message": "Hello world!"
    },
    {
        "name": "Bob",
        "message": "Hey, Andrew!"
    }
]
```

然后, 在each循环中, 你可以看到{{message}}` {{name}}`, 这会引用 每一个数组元素的值来替代(Andrew 和 Bob 替换 name, 以及各自的问候信息.)

当返回到你的浏览器, 你还看不到任何的改变. 因为讯息数组还没被传送到模板, 所以Meteor遍历不到任何东西来呈现.

你的`chat.html`最后应该是这样的

```html
<head>
  <title>chat</title>
</head>
<body>
  \{\{> entryfield}}
  \{\{> messages}}
</body>
<template name="entryfield">
    <input type="text" id="name" placeholder="Name" /> <input type="text" id="message" placeholder="Your Message" />
</template>

<template name="messages">

    <p>
		\{\{#each messages}}
            <strong>{{name}}</strong>- {{message}}<br/>
        \{\{/each}}
    </p>
</template>
```
##### The Javascript
从现在开始, 我们处理的大部分代码都是客户端代码, 所以, 除非特别说明, 以下的代码都是在`if (Meteor.is_client)`代码块中.

在我们编写展示讯息的代码之前,让我们先新建一个`Collection`. 从本质上讲, 这是一组Models. 换句话说, 在这个chat应用的环境下, Messages collection保存着整个聊天记录, 而每条讯息记录是一个Model.

在if语句前, 添加如下代码来初始化Collection:

```javascript
Messages = new Meteor.Collection('messages');
```

因为我们希望这个Collection可以在客户端和服务端被创建, 所以我们把它写在了客户端代码块之外.

由于Meteor为我们做了大部分的工作, 要展示聊天记录是非常容易的. 只需要把下面的代码添加进if语句里边.

```javascript
Template.messages.messages = function(){
    return Messages.find({}, { sort: { time: -1 }});
}
```

让我们拆开来分析这段代码:

**Template**.messages.messages = function(){ … }

第一部分`Template`表示我们正在修改一个模板的行为.

Template.**messages**.messages = function(){ … }

第二部分`messages`是模板的名字, 表示是在修改哪一个模板.

例如,如果我们想要对"entryfield"模板做些什么,   只需把代码改成`Template.entryfields.variable = function(){ … }`(在这里, 请别这么做)

Template.messages.**messages** = function(){ … }

第三部分的这个`messages`代表的是一个这个模板里的一个变量. 还记得我们的each循环遍历`messages`吗? 这就是那个mesaages.

当你打开浏览器时, 页面还是没有什么改变. 这是意料之中的事, 因为我们只抓取的讯息, 而没有展示出来.

此时,你的`chat.js`应该是这样的. 是否很惊讶就只需在服务器写这么些代码我们就能展示一个实时的聊天记录应用.

```javascript
Messages = new Meteor.Collection('messages');

if (Meteor.is_client) {
  Template.messages.messages = function(){
    return Messages.find({}, { sort: { time: -1 }});
  }
}
```
##### Adding a Message through the Console
这部分的内容是可选的, 当然它有助于你调试程序. 你可以直接跳过往下学习建立form来响应key press.

如果你想要测试你的讯息显示代码, 你可以手动插入一条记录到数据库. 打开你的浏览器控制台, 并输入如下:

```
Messages.insert({ name: 'Andrew', message: 'Hello world!', time: 0 })
```

这将会在数据库中新建一条记录, 如果正确的操作了的话,那浏览器就会即刻更新这条讯息在页面上.

##### The Message Entry Form

返回到`chat.js`, 我们即将要完成的是允许用户在输入框中提交聊天讯息到数据库.
在if语句里头最下面添加下面这段代码

```javascript
Template.entryfield.events = {
  "keydown #message": function(event){
    if(event.which == 13){
      // Submit the form
      var name = document.getElementById('name');
      var message = document.getElementById('message');

      if(name.value != '' && message.value != ''){
        Messages.insert({
          name: name.value,
          message: message.value,
          time: Date.now()
        });

        name.value = '';
        message.value = '';
      }
    }
  }
}

```

这段代码比较多, 让我们一步步分析. 你应该还有印象, 紧跟`Template`之后的词表示我们即将要修改的template的名字. 跟前面修改的模板是`messages`不一样的是, 这里我们要修改的是`entryfield`.
template的`event`属性会包含有一个对象, 这个对象的keys的格式如下:

```
"[eventname] [selector]"
```
比如, 如果我们想绑定了一个`click`事件给了ID为`hello`的按钮. 则需要像下面这样添加一个`events`对象

```
"click #hello": function(event){ … }
```

在我们这个例子中,我们绑定了一个`keydown`事件给了ID为`message`的输入框. 这个输入框在我们一开始的`chat.html`已经建好.

在`events`对象里, 每一个key都有一个函数作为它的值. 当事件被触发的时候, 这个`event`对象会被作为第一个参数传给这个函数.在我们的chat应用, 每一次在输入框中按下一个按键(`keydown`), 这个函数都会被调用.

在函数里的代码非常简单. 首先, 我们先要检测按下的是否"enter"键("enter"的keycode 是13). 接着,通过ID来得到两个输入框中的DOM元素. 最后, 我们检查确保输入框中的值不为空, 用户不允许发送空的名字和讯息.

下面的代码需要注意了, 它会被直接插入到数据库中

```javascript
Messages.insert({
  name: name.value,
  message: message.value,
  time: Date.now()
});
```
你会发现,这其实跟我们直接在浏览器控制台中输入的代码很相似, 只是我们用DOM元素的值来替换固定值. 另外, 欧文木讷添加了时间值来确保是按照时间来排序.

最后,我们把两个输入框设置为空`''`

现在, 你可以打开浏览器尝试在输入框输入名字和讯息. 按下"enter"后,输入框会被被清空, 而且这则讯息会立马出现在你的输入框下边. 使用另外一个浏览器窗口或者其他浏览器打开同样的链接(`http://127.0.0.1:3000`).	尝试输入一则新的讯息, 此时你就会看到Meteor的强大之处.无须单独写一行代码来更新讯息记录, 就可以实时同步在不同的客户端浏览器中.

##### Conclusion

While Meteor is pretty cool to work with and there are some pretty useful applications for it, like Derby.js, it is immature. For examples of this, just browse through the documentation and look for the red quotations. For example, the documentation states the following about MongoDB collections:

>Currently the client is given full write access to the collection. They can execute arbitrary Mongo update commands. Once we build authentication, you will be able to limit the client's direct access to insert, update, and remove. We are also considering validators and other ORM-like functionality.

对于任何一个产品而言, 用户拥有数据库的所有读写权限都是非常大的问题, 非常危险.


原文来自Andrew Munsell [Introduction to Realtime Web with Meteor and Node.js](http://www.andrewmunsell.com/blog/introduction-to-realtime-web-Meteor-and-nodejs/#.UTyJZtHN-tv)

另附上Tom的 Vimeo screencast(需FanQiang)
http://av.vimeo.com/31624/419/93304834.mp4?aksessionid=d86b5b612c86d70bd2022ae1050aa1be&token=1362992353_ce10e413b18fa6f2d0e3483ef6b82894 640 320 http://b.vimeocdn.com/ts/278/431/278431171_960.jpg