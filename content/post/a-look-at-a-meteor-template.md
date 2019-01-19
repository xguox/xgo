---
title: "A Look at a Meteor Template"
date: 2013-03-06T16:01:23+08:00
draft: false
tags: ["Translation", "JavaScript"]
categories: ["Translation", "JavaScript"]
---

## LEVEL: BEGINNER

学习一门语言或框架的最好方法之一是从零开始开发一个应用程序.

但是,当你开发了N次的To-do list应用之后, 或许你会想要一些稍微新鲜的玩意. 所以这一次, 让我们来看看一些实际的应用的代码, 这里我们用的是 [Telescope](http://telesc.pe/)

Telescope是一个基于Meteor的开源社交新闻网站. [Reddit](http://reddit.com/) or [Hacker News](http://news.ycombinator.com/)的实时版本.

我们即将看到的代码是简化版的`post_edit`[模板](https://github.com/SachaG/Telescope/blob/master/client/views/posts/post_edit.html)及其[控制器(controller)](https://github.com/SachaG/Telescope/blob/master/client/views/posts/post_edit.js). 听名字你应该就知道, 这个模板的作用是提供一个简单的表单来编辑一篇已经存在的post. post有标题(title), 链接(URl), 主体(body), 以及一个或多个的类别(categories). 该程序实际的表单中还会有一些额外的东西, 但我们现在不需要关心那些.

![](http://theMeteorbook.com/images/telescope-post-edit.png)

这篇文章的目的是让你对Meteor的模板有个大概的认识. 如果你在其他的环境下使用过模板系统就会感到很熟悉, 如果没的话,那就现在开始学习吧.

## Basic Organization

Meteor模板系统有两个部分组成,分别是: HTML模板和JavaScript控制器. HTML部分只是纯粹的有变量占位符的哑巴模板, 所有的实际动作都是通过JavaScript产生的.

顺便说句, Meteor通过`template`标签的`name`属性来得到这个模板的名字(`<template name="post_edit">`), 因此,建议使用同一名字命名所有相关的文件和模板本身, 如在这个例子中的`post_edit.html`和`post_edit.js`

post_edit.html template


```html
<template name="post_edit">
  \{\{#with post}}
    <form>
      <div>
        <label>Title</label>
        <input id="title" type="text" value="{{headline}}" />
      </div>
      <div>
        <label>URL</label>
        <input id="url" type="text" value="{{url}}" />
      </div>
      <div>
        <label>Body</label>
        <textarea id="body" value="" class="input-xlarge">{{body}}</textarea>
      </div>
      <div>
        <label>Category</label>
        \{\{#each categories}}
          <label class="checkbox inline">
            <input id="category_{{_id}}" type="checkbox" value="{{_id}}" name="category" {{hasCategory}} /> {{name}}
          </label>
        \{\{/each}}
      </div>
      <div class="form-actions">
        <a class="delete-link" href="/posts/deleted">Delete Post</a>
        <input type="submit" value="Submit" />
      </div>
    </form>
  \{\{/with}}
</template>

```


post_edit.js controller


```javascript
Template.post_edit.helpers({
  post: function() {
    return Posts.findOne(Session.get('selectedPostId'));
  },
  categories: function() {
    return Categories.find().fetch();
  },
  hasCategory: function() {
    var post = Posts.findOne(Session.get('selectedPostId'));
    return _.contains(post.categories, this.name) ? 'checked' : '';
  }
});

Template.post_edit.events = {
  'click input[type=submit]': function(e) {
    e.preventDefault();

    var selectedPostId = Session.get('selectedPostId');
    var post = Posts.findOne(selectedPostId);

    var categories = [];
    $('input[name=category]:checked').each(function() {
      categories.push($(this).val());
    });

    var properties = {
      title:         $('#title').val(),
      url:           $('#url').val()
      body:          $('#body').val(),
      categories:    categories,
    };

    Posts.update(selectedPostId, {$set: properties}, function(error) {
      if (error) {
        alert(error.reason);
      }
    });
  },
  'click .delete-link': function(e) {
    e.preventDefault();
    if(confirm("Are you sure?")) {
      var selectedPostId = Session.get('selectedPostId');
      Posts.remove(selectedPostId);
    }
  }
};

```

代码看起来很多, 但是只要拆开来看,即使你从未看过Meteor的代码也会觉得很容易.

## Handlebars Tags

首先来快速浏览下HTML部分. 你一定注意到这种奇怪的标签用法了吧. 这些都是[Handlebars](http://handlebarsjs.com/)标签用来让我们展示动态数据.
先来看第一个 handlebars 标签 `\{\{#with post}}`

The "with" block helper


```html
<template name="post_edit">
  \{\{#with post}}
    <!-- do something with "post" -->
  \{\{/with}}
</template>

```

[block helper](http://handlebarsjs.com/block_helpers.html)是一种快捷的方式告诉我们要在哪个模板对象上工作. 在`\{\{#with post}}`block里, 我们的应用知道`this`代表一个`post`. `this`也是传说中方法的上下文.

## Template Helpers

你可能会好奇,我们是从哪里得到`post`, 找到你的controller里的


```javascript
//The "post" template helper

Template.post_edit.helpers({
  post: function() {
    return Posts.findOne(Session.get('selectedPostId'));
  }
});

```

在这我们要做的是添加一个`post`[Template Helper](http://docs.Meteor.com/#template_helpers)到我们的`post_edit`模板. Template helpers 是一些简单的JavaScript方法允许你使用 Handlebars template.

在这个例子中,不管任何时候我们的Handlebars模板调用`post`, 我们都能在我们的`Posts` collection 中找到current post并返回之. 要做到这样首先我们需要用到`selectedPostId`这个通过路由设置的Session变量(but that's for another lesson)

## Object Properties

继续往下, 我们将看到第二个handlebars标签`{{title}}`


```html
//The "title" template helper
<div>
  <label>Title</label>
  <input id="title" type="text" value="{{title}}" />
</div>

```

尽管不能用什么方式告诉你这个`{{title}}`不是一个Template helper, 而是我们的`post`对象的属性之一. 无须任何其他的工作,Meteor很神奇地提供给了我们使用的.

## Nested Objects

链接(URL)和主体(Body)表单域用的都是相同的模式, 而类别(Category)域则有些不同了.在这里, 我们希望通过类别检查展示一系列复选框给所有网站的预定义类别.

首先的工作是准备一系列的类别选择, 我们通过`\{\{#each categories}}`这个block helper来完成:

Looping over categories with the "each" helper


```html
<div>
  <label>Category</label>
  \{\{#each categories}}
    <label class="checkbox inline">
      <input id="category_{{_id}}" type="checkbox" value="{{_id}}" name="category" {{hasCategory}} /> {{name}}
    </label>
  \{\{/each}}
</div>

```
这个helper完成了两件事: 遍历`categories`对象, 以及在代码块里使用`this`来表示每一个遍历结果.

或许你已经猜到, `categories`像`posts`那样是一个Template helper. 在这个例子中,我们只返回了在我们的数据库中的一系列类别(categories)

The "categories" template helper


```javascript
  categories: function() {
    return Categories.find();
  }

```
还记得我说过block helper 会改变`this`的值吗?这就是为什么我们能看到`{{_id}}`而不会混乱. 因为这事在block helper里边, 因此可以知道,我们处理的是category的ID,而不是post的ID

## More Helpers

到目前为止, 我们的helper简单地查询了我们的collections并返回了数据. 但helper却不仅限于此. 让我们一起看一看 `{{hasCategory}}`这个helper

The "hasCategory" template helper


```javascript
hasCategory: function() {
  var post = Posts.findOne(Session.get('selectedPostId'));
  return _.contains(post.categories, this.name) ? 'checked' : '';
}

```

还记得吧, 我们是在block `\{\{#each categories}}`里调用了这个helper, 这就意味着这里`this`表示的是category

那么我们现在是首次得到我们的current post对象, 然后检查看current category是否包含在这个post的categories里.如果包含有, 则我们很容易地返回"checked"字符串用以标记我们这个复选框被选中.

你可能已经注意到, `_.contains()`是一个 [Underscore](http://underscorejs.org/)方法. Underscore是一个非常有用的JavaScript工具包并被内置绑定到了Meteor里边.

## Template Events

我们现在的代码足够用来加载一篇post的标题,URL,主体,以及类别了. 但还是需要一种保存这个表单的方法. 也就是下面的模板事件(template events)

Template events


```javascript
Template.post_edit.events = {
  'click input[type=submit]': function(e) {
    // do something when the users clicks input[type=submit]
  },
  'click .delete-link': function(e) {
    // do something when the users clicks .delete-link
  }
};

```
如你所见, 定义 template events其实就是简单把事件和CSS选择器放在一起. 在这里我们有两个处理监听,一个是提交表单,另一个是删除post.
先来看看表单提交处理:

Our form submit handler


```javascript
'click input[type=submit]': function(e) {
  e.preventDefault();

  var selectedPostId = Session.get('selectedPostId');
  var post = Posts.findOne(selectedPostId);

  var categories = [];
  $('input[name=category]:checked').each(function() {
    categories.push($(this).val());
  });

  var properties = {
    title:         $('#title').val(),
    url:           $('#url').val()
    body:          $('#body').val(),
    categories:    categories,
  };

  Posts.update(selectedPostId, {$set: properties});
},

```

处理方法带有两个参数, event 和 template 实例. 现在, 我们可以忽略template 实例. 使用`e.preventDefault();`来开始你的处理可以很好的阻止浏览器执行默认的事件行为.

如你所见, 这里有大量的`$`符号. 除了Underscore, Meteor还绑定了jQuery. 看吧, 你已经了解了Meteor的一半了.

再来一次, 处理categories相较于其他的来说比较麻烦一些.  通过使用jQuery, we're checking for checked checkboxes(敢不敢把这句话说上10次,LOL)并使用他们的ID来填充给一个数组.

接下来我们就能够建立我们的属性对象, 并使用Meteor的[update()](http://docs.Meteor.com/#update)可以简单地提交我们的修改给数据库.

删除post更是简单
Our post delete handler


```javascript
'click .delete-link': function(e) {
  e.preventDefault();
  if(confirm("Are you sure?")) {
    var selectedPostId = Session.get('selectedPostId');
    Posts.remove(selectedPostId);
  }
}

```
只需要简单地把要删除的ID传送给Meteor的[remove()](http://docs.Meteor.com/#remove)方法就搞定.

## Other Callbacks

Meteor给管理模板提供了一些其他的回调, 比如[created](http://docs.Meteor.com/#template_created)和[render](http://docs.Meteor.com/#template_rendered)回调

尽管我们在这里没有使用到, 但只要Meteor完成渲染一个模板, 那么`rendered`回调可以很容易的通过jQuery来做一些DOM操作.

## Conclusion

CRUD (Create, Read, Update, Delete)操作经常就是组成网站的主要功能. Meteor能如此简单的完成这些工作显然是要大加分的.

更重要的是, Meteor代码所依赖的很多模式和库都是你所熟悉的.这意味着可以非常快的掌握,只需要一些JavaScript知识.

当然,随着您的需求的发展,你需要深入挖掘这个框架的复杂性.但是即使你只知道Meteor最基本的功能你也同样能作出一些东西来, 我想,这大概就是为什么Meteor非常好玩的原因之一.

原文来自 Sacha Greif  [A Look at a Meteor Template](http://theMeteorbook.com/2013/02/20/a-look-at-a-Meteor-template/)












