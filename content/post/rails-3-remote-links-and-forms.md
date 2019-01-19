---
title: "Rails 3 Remote Links and Forms: A Definitive Guide"
date: 2012-07-11T16:01:23+08:00
draft: false
tags: ["Translation", "Ruby"]
categories: ["Translation", "Ruby"]
---


##### blahblah:
> 这些天遇上一些困难请教Larry的时候他没少给我一些英文的资料,虽然能够阅读,但是始终觉得翻译回中文以后再看也方便些.不过有些地方翻的比较蹩脚呀.

> 原文:[Rails 3 Remote Links and Forms: A Definitive Guide](http://www.alfajango.com/blog/Rails-3-remote-links-and-forms/)

Spoiler Alert:如果你喜欢魔法,请停止阅读这篇文章.因为,让Rails的表单、连接、输入等如此强大的Rails 3 UJS驱动—-Rails.js,当你知道它的工作原理后,你就知道其实它没有多么的神奇.

##### Rails.js做什么

1. 它找到远程链接、表单和输入,并通过Ajax覆盖这些的click事件以提交到服务器端.
2. 它触发6种javascript事件来让你绑定一些回调函数用以处理和操纵这个Ajax响应
3. <del>它处理从服务器端得到的Ajax回应</del>

##### Rails.js不做什么

请注意到上述最后一点是加了删除线的.当启用新的Rails3远程功能时候,这似乎是混乱的最大来源.因为已经根深蒂固的习惯于Rails2的 link_to_remote 与 remote_form_for,我们期待Rails3也同样的会为我们的表单、链接处理Ajax响应.但是Rails3却没有这么做,它把这些都交给你去完成. Rails会做的是帮你把包裹搬上房间,但不会帮你打开包裹.This is by design;为什么你想要侍从帮你使用你的东西呢?同样的,每个元素的数据实际处理方式很大程度上是独一无二的,并且取决于你再各种情况下工作所得的的数据.所以,Rails把这一部分留给你来做决定,而我们则回到了过去.

##### HTML5的角色

你可能已经听说Rails3将使用HTML5.的确是的.但HTML5所扮演的实际角色并没有你想象中的那么重要. 是否还记得Rails.js所做的第一样东西.在它能覆盖所有远程元素的click事件和submit事件前,它需要先找到这些所有的元素. 因此,我们需要一种方法来指定我们的远程元素.在以前,我们可能会添加 class="remote" 到我们的远程元素.但现在,所有的class属性都跟样式相关联了.难道就没有一种办法能区分出远程元素而不用污染到我们的CSS类. 这,在HTML5中是有的.有这么一段HTML5规范:现在,你可以添加任意标签到一个HTML元素,只要这个标签的起始部分是"data-".因此,下面这是一个有效的HTML标记:

```html
<a hre="stuff.hml" data-rocketsocks="whateva"]] > Blast off!</a]] >
```

Rails 3充分利用了这种新的HTML标记,它通过在HTML元素中添加 :remote => true 来转换所有的链接和表单使其拥有了标签 data-remote = true

```html
<%= link_to "Get remote sauce", {:action => "sauce"}, :remote => true, :class => "button-link" %># => <a href="sauce" class="button-link" data-remote=true>Get remote sauce</a>
```

现在Rails.js通过如下的选择器找到所有的远程链接:

```javascript
$('form[data-remote]')
$('a[data-remote],input[data-remote]'
```

并且分别通过一个ajax请求到form的action和link的href属性从而重载了submit和click动作, Then it does the same thing wirh remote forms and inputs, using the form's action. 就是这样,HTML5只是提供一个方便、语法、方式给我们用以指定及选择劫持哪个元素(???)

##### 处理Ajax响应

那么,我们到底是如何根据ajax响应做一些事情呢?幸运的是,当Rails.js远程发送你的请求时,它同时也触发了6个自定义事件,并传输相应的数据/响应到每一个事件.你也可以绑定你自己的处理函数到这些事件. 这6个事件是:

```ruby
ajax:before
ajax:loading
ajax:success
ajax:failure
ajax:complete
ajax:after
```
##### Update:

>在这篇文章写下后,Rails团队对 jQuery UJS driver 做了些改动.在最新版本的Rails.js中,只剩下了以下4个回调事件:

```ruby
ajax:beforeSend
ajax:success
ajax:complete
ajax:error
```

这样,在你的页面中,你将会像下面这样绑定这些事件到一个函数中:

```javascript
$('.button-lnk').bind('ajax:success', function(){  alert("Success!");
});
```
你会发现,通过绑定一些函数到实体/事件,在Rails中的所有的javascript功能都是很方便的.

Binding is good,因为它不是引人注目的.javascript功能和html标记分别作为完全独立单位但通过javascript绑定在一起的存在.如果用户没有用到javascript,那么javascript所绑定到html实体的js函数将永远不会被执行,用户所得到的,只是干干净净、有效的HTML

##### 把他们都放在一起

解释的足够多了,下面让我们建立一个加载一些内容到页面的远程表单.我们会提供给用户即时反馈,完整地处理所有的错误,以及能让用户重置表单再做一次.有了以上知识的武装,希望下面的没有人会看起来以为是魔法. 接下来,我们将假定在我们的Rails app中,有一个Comment model,且该model在数据库中有一个text类型的"content"字段.通过提交一个远程表单,我们能让用户创建一个comment,并将这个comment插入到页面中.

>你可能注意到,我们直接请求了一个html的响应,但却以JSON的形式返回一些错误.我们使用 respond_with 覆盖了许多魔术方法.这一切说明我们在请求/响应中有多大的控制权.

>接着的Part2将通过一个更production-appropriate的例子叙述了如何请求一个JS(xml,json,文本或其他)的响应.

##### View

```ruby
<%= form_for @comment, :remote => true, :html => { :'data-type' => 'html', :id => 'create_comment_form' } do |f| %>  <%= f.text_area(:content) %>
  <div class="validation-error"></div>
  <%= f.submit %>
<% end %>

<div id="comments"></div>
```

##### Controller

```ruby
respond_to :html, :xml, :json...

def create
  @comment = Comment.new( params[:comment] )

  if @comment.save
    respond_with do |format|
      format.html do
        if request.xhr?
          render :partial => "comments/show", :locals => { :comment => @comment }, :layout => false, :status => :created
        else
          redirect_to @comment
        end
      end
    end
  else
    respond_with do |format|
      format.html do
        if request.xhr?
          render :json => @comment.errors, :status => :unprocessable_entity
        else
          render :action => :new, :status => :unprocessable_entity
        end
      end
    end
  end
end
```

显而易见的,我们需要一个显示comment的partial模板 `_show.html.erb` .希望你知道如何做到.

在这一点上,我们有一个提交到远程服务器上的表单,并且我们的服务器响应以我们在view中的partial,并准备插入到页面中去.但有一点是,它并不是真正的被插入到页面中.

##### Javascript

是时候绑定一些处理函数到那些触发的ajax事件.因此,在我们的表单页面中,我们将希望加载这个javascript(可能会在标签或者在一个分离开的js文件)

>Update:

>下面这段已经更新以绑定到最新版本的Rails.js所允许的回调.

```javascript
$(document).ready(function(){
  $('#create_comment_form')
    .bind("ajax:beforeSend", function(evt, xhr, settings){
      var $submitButton = $(this).find('input[name="commit"]');

      // Update the text of the submit button to let the user know stuff is happening.
      // But first, store the original text of the submit button, so it can be restored when the request is finished.
      $submitButton.data( 'origText', $(this).text() );
      $submitButton.text( "Submitting..." );

    })
    .bind("ajax:success", function(evt, data, status, xhr){
      var $form = $(this);

      // Reset fields and any validation errors, so form can be used again, but leave hidden_field values intact.
      $form.find('textarea,input[type="text"],input[type="file"]').val("");
      $form.find('div.validation-error').empty();

      // Insert response partial into page below the form.
      $('#comments').append(xhr.responseText);

    })
    .bind('ajax:complete', function(evt, xhr, status){
      var $submitButton = $(this).find('input[name="commit"]');

      // Restore the original submit button text
      $submitButton.text( $(this).data('origText') );
    })
    .bind("ajax:error", function(evt, xhr, status, error){
      var $form = $(this),
          errors,
          errorText;

      try {
        // Populate errorText with the comment errors
        errors = $.parseJSON(xhr.responseText);
      } catch(err) {
        // If the responseText is not valid JSON (like if a 500 exception was thrown), populate errors with a generic error message.
        errors = {message: "Please reload the page and try again"};
      }

      // Build an unordered list from the list of errors
      errorText = "There were errors with the submission: \n<ul>";

      for ( error in errors ) {
        errorText += "<li>" + error + ': ' + errors[error] + "</li> ";
      }

      errorText += "</ul>";

      // Insert error list into form
      $form.find('div.validation-error').html(errorText);
    });

});
```

>这些函数每一个都能轻易地抽象化,这样他们能轻易地或者自动地应用在我们所有的远程表单中.

这些绑定也以同样的方式工作于远程链接中.但我想我们已经使用了一个远程表单在这个例子中,由于这需要一些额外的但没有明显区别的步骤(例如清除表单或增添验证错误)

同时,需要注意到:从ajax事件所传输到你的函数的数据并不是在同一个顺序.在ajax:success中是 evt,data,status,xhr ,而在ajax:failure中是,eve,xhr,status,error.This will get ya every time.

##### 别急,还有更重要的

如果你从文章开始就做足功课,并且看过了Rails.js文件,你应该会注意到一些额外的细节.

##### .live

Rails.js实际上用 .live() 而不是直接绑定click事件到远程表单、链接、输入.这样,在上面的javascript中,你可以替换 .bind() 为 .live() 显得更文艺一些.

##### Update:

>如果要在你的ajax处理中使用 .live() ,请确定你所使用的jQuery版本>v1.4.4,因为早些的版本会在IE中出现一些问题.

##### :confirm => "R U sure? "

你可能还会注意到 Rails.js文件处理 :confirm => "R U sure? "(当你点击或者提交一些东西的时候嘣的弹出一个小窗口并显示"R U sure"(你也可以只写 :confirm => true 使用默认的信息)),就像远程功能,Rails 3简单地添加有效的标签 data-confirm=true 到你的HTML元素

##### :disable_with => "Submitting…"

Rails 3给了我们另外的一个选项,:disable_with,这样当一个远程表单被提交的时候, we can give to form input elements to disable and re-label them.添加data-disable-with标签到这些输入中,使Rails.js能选择和绑定这个功能.

想知道如何让远程表单或者链接工作于 js.erb(或者其他任何与之相关联的形式),请看这篇文章的[Part 2](http://xguox.me/rails-3-remote-links-and-forms-part-2-data-type-with-jquery.html)