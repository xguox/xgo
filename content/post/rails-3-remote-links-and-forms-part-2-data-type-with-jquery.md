---
title: "Rails 3 Remote Links and Forms Part 2: Data-type (With jQuery)"
date: 2012-07-12T16:01:23+08:00
draft: false
tags: ["Translation", "Ruby"]
categories: ["Translation", "Ruby"]
---

自从写了[Rails 3 Remote Links & Forms Definitive Guide](http://www.alfajango.com/blog/Rails-3-remote-links-and-forms/)有一个问题也接踵而至:

#### 我们该如何使远程链接或者表单取回 js.erb而不是一个html的partial?

在上一篇中,我们请求一个html partial并通过ajax回调插入到页面中.但如果我们想要javascript被执行?或者XML、JSON被解析?又或者显示出纯文本呢?

## Equal parts Rails & jQuery

首先,我们需要明白Rails3中 :remote => true 的过程. It's equal parts Rails and jQuery magic.但表担心,这只是一个小魔术,捆绑成4个步骤的过程: 1. Rails告诉jQuery,"哥们,把你的ajax绑定到这霸气的表单/链接",通过 :remote => true 这个选项. 2. jQuery劫持?元素的click/submit事件并绑定到.ajax()方法.现在,元素提交通过使用ajax取代在浏览器中加载一个新页面. 3. 当这个元素被click/submit时,Rails接收到ajax网页请求并响应以一些内容. 4. jQuery接收到响应的内容有.ajax()方法hi-jacked在我们的的元素中,并为我们提供回调用以在页面中处理这个响应.

在这篇文章中,我们正在探索各种各样的、使我们能够指定ajax的响应格式并据此做出处理的方式,这实际上涵盖上述所有4个步骤.

## 步骤 1 & 2:设置data-type

当浏览器发送web请求,在浏览器到服务器,服务器返回浏览器的过程中,部分 请求/响应 的header能够指定内容的格式.当在浏览器中加载一个页面,内容的类型通常从url的拓展可以推断得出.尽管如此,jQuery能够直接在ajax请求的header部分设置我们所想要的data-type.

## jQuery允许dataType参数

jQuery的.ajax()方法提供一个可选的dataType参数用以指定我们所想要的响应data-type.这允许jQuery在请求的HTTP Accept header指定格式类型,然后封装响应的内容到适当的数据对象使得更容易操作.

>在jQuery1.4中,如果你不指定响应的data-type,jQuery会检查响应的MIME类型头并智能的猜测其为data-type,动态的转换响应对象的data-type

在.ajax()文档页面有更多的信息,但基本的类型有如下:

<table>
  <tbody><tr>
    <th>dataType</th>
    <th>Behavior</th>
  </tr>
  <tr>
    <td>"xml"</td>
    <td>返回一个能呗jQuery处理的XML文档</td>
  </tr>
  <tr>
    <td>"html"</td>
    <td>返回纯文本的html,调用标记中的所有&lt; script&gt;标签</td>
  </tr>
  <tr>
    <td>"script"</td>
    <td>Evaluates the response as JavaScript并以纯文本格式返回</td>
  </tr>
  <tr>
    <td>"json"</td>
    <td>Evaluates the response as JSON 并返回一个javascript对象</td>
  </tr>
  <tr>
    <td>"jsonp"</td>
    <td>在一个JSON块中使用JSONP加载响应</td>
  </tr>
  <tr>
    <td>"text"</td>
    <td>返回一个纯文本字符串</td>
  </tr>
</tbody></table>

## Rails.js设置dataType参数

Rails UJS驱动通过我们在远程链接/表单中指定的data-type属性来设置我们的ajax dataType参数.如果我们不明确地指定data-type,那默认的data-type将使用公有的$.ajaxSettings.如果我们没有提前设置它,那么一个通用的请求发送出后将会收到任意类型的响应.

```javascript
dataType = element.attr('data-type') || ($.ajaxSettings && $.ajaxSettings.dataType);
```


老版本的UJS驱动会默认一个在script 中的data-type,而不是发送一个通用的请求.看上去,这像是一个明智的做法,但如果在我们controller的action中没有定义format.js的话, Rails将会抛出一个异常. 而在较新版本的UJS驱动中则简单地使用jQuery的'/'默认dataType.这会告诉服务器,"不管你收到什么都给我".然而,这会让controller响应以在Responder中所列出的第一种格式(见下一节).这样的话,如果 format.html 列在 format.js 前面的话,app会响应以HTML格式(这意味着将会尝试跳转到POST或DELETE方法的ajax请求).这并不是我们想要的.

所以在最新版本的UJS驱动中,我们找到如何设置默认情况,比如说,当它告诉服务器,"尽管我更希望得到JS格式,但我会得到所有你得到的."这样,如果所有可用的响应格式包括format.js都被定义,那么返回的将会是JS格式.当然,如果format.js没有被定义,那controller将会继续按照列表顺序返回第一个.

```javascript
// Simplified for clarityif (dataType === undefined) {
  xhr.setRequestHeader('accept', '*/*;q=0.5, text/javascript');
}
```

## 步骤 3 :在Rails的controller中的响应
现在我们已经有了一个ajax请求,同时,在该请求的header指定我们希望得到的data-type.我们的Rails app接收到这个请求,就会路由到相应的action,并渲染出一个响应.

我们的controller决定渲染些什么内容作为响应,以及是什么样的格式.respond_with 和 respond_to 这两个在Rails的Responder类中的方法会检查请求的 Accept 头(通过dataType设置)以渲染出适当的响应.

对于基于对象的data-types,如XML和JSON,我们通常会对其序列化并返回一个指定格式的对象.这就是Responder默认所做的.而对于给予内容的data-types,如JS或html,我们通常会渲染一个 js.erb 或 html.erb 文件.

>技术上,我们也能有一个自定义的 json.erb 或 xml.erb模板来渲染一个自定义的数据对象.Responder会查找这些模板,如果存在则渲染之.

## 步骤 4 : jQuery处理响应
从上一篇文章可知,我们绑定了我们处理响应的jQuery代码在ajax:success,ajax:error,ajax:complete这些回调中.

现在,我们对于各种不同的data-types及如何设置它们有了新的认识,我们能够修改我们的响应回调来处理我们所想要的data-type.在上一篇文章中,我们的回调绑定处理了一个HTML响应.

处理HTML响应:

```javascript
$('#i-want-html')  .bind('ajax:success', function(evt, data, status, xhr){
    var $this = $(this);

    // Append response HTML (i.e. the comment partial or helper)
    $('#comments').append(xhr.responseText);

    // Clear out the form so it can be used again
    $this.find('input:text,textarea').val('');

    // Clear out the errors from previous attempts
    $this.find('.errors').empty();

  })
  .bind('ajax:error', function(evt, xhr, status, error){

    // Display the errors (i.e. an error partial or helper)
    $(this).find('.errors').html(xhr.responseText);

  });
```

同样的,我们能很轻易的修改上面的代码来处理JSON响应:

处理JSON请求:

```javascript
$('#i-want-json')  .bind('ajax:success', function(evt, data, status, xhr){
    var $this = $(this);

    // do something with 'data' response object

    $this.find('input:text,textarea').val('');
    $this.find('.errors').empty();
  })
  .bind('ajax:error', function(evt, xhr, status, error){
    var responseObject = $.parseJSON(xhr.responseText),
        errors = $('<ul />');

    $.each(responseObject, function(){
      errors.append('<li>' + this + '</li>');
    })

    $(this).find('.errors').html(errors);
});
```

最后,如果是要处理一个JS请求,则我们不需要绑定任何的回调函数.记住,如果是 data-type: 'script' ,jQuery会自动执行页面中的JavaScript响应.

处理JS请求:

```

//Nothing!
```

如果 我们的响应没有自动为我们执行,那我们的处理方式可能是像下面这样:

```javascript
$('#i-want-js').bind('ajax:complete', function(evt, xhr, status){  eval(xhr.responseText);
});
```

注意,我们已经绑定了回调 ajax:complete 来取代ajax:success|error.这是因为,我们的success/error处理都在js.erb文件.

>上面的处理方式并不非完全和自动执行函数一致.上述的函数会在 $('#i-want-js') 元素的上下文执行我们的脚本,同时,jQuery在 $(window) 上下文自动执行我们的脚本.

现在,让我们来个总结

## 使用js.erb的例子

就像上一篇文章,我们会创建一个comment并通过ajax提交.所不同的是,这次我们会响应以 js.erb

在controller中,我们会添加一些响应html、hs、json请求的功能.

comments_controller.rb

```ruby
class TestCommentsController < ApplicationController  respond_to :html, :js
  ...
  def create
    @comment = Comment.new( params[:comment] )

    flash[:notice] = "Comment successfully created" if @comment.save
    respond_with( @comment, :layout => !request.xhr? )
  end
end
```

如果你对Rails3的Responder不熟悉,下面的代码效果是一样的:

```ruby
class TestCommentsController < ApplicationController ...
  def create
    @comment = Comment.new( params[:comment] )

    respond_to do |format|
      if @comment.save
        flash[:notice] = "Comment successfully created"
        format.html { redirect_to(@comment), :layout => !request.xhr? }
        format.js { render :js => @comment, :status => :created, :location => @comment, :layout => !request.xhr? }
      else
        format.html { render :action => "new", :layout => !request.xhr? }
        format.js { :js => @comment.errors, :status => :unprocessable_entity }
     end
  end
end
```

现在,我们来设定index页面来列出所有的comments,以及包括我们的ajax表单用以创建一个新comment.我们不需要在我们的表单中写任何data-type这鸽HTML5属性,Rails UJS会默认我们需要JS

index.html.erb

```html

<div id="comments"></div>
<%= form_for :comment, :remote => true, :html => { :id => 'new-comment-form' } do |f| %>
  <%= f.text_area(:body) %>
  <div class="errors"></div>
  <%= f.submit %>
<% end %>
```

最后,我们需要创建 js.erb模板,该模板会被执行并插入响应到我们的页面中.

create.js.erb

```javascript
var el = $('#new-comment-form');
<% if @comment.errors.any? %>

  // Create a list of errors
  var errors = $('<ul />');

  <% @comment.errors.full_messages.each do |error| %>
    errors.append('<li><%= escape_javascript( error ) %></li>');
  <% end %>

  // Display errors on form
  el.find('.errors').html(errors);

<% else %>

  // We could also render a partial to display the comment here
  $('#comments').append("<%= escape_javascript(
      simple_format( @comment.body )
    ) %>");

  // Clear form
  el.find('input:text,textarea').val('');
  el.find('.errors').empty();

<% end %>
```

## 写在最后!

重申一次,这里我们不需要绑定任何ajax回调.It just works.

>有时候,javascript响应不会像它本应该的那样执行,取而代之的是作为一个字符串返回.这通常意味着有在这个响应的javascript中有一些异常.这是很烦人的,但它不会从自动执行响应中抛出任何可见的javascript错误.

See Rails js.erb Remote Response not Executing.