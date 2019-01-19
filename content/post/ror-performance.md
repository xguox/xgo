---
title: "给 Ruby on Rails 提速"
date: 2011-12-25T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

[Repost](http://www.ibm.com/developerworks/cn/opensource/os-Railsn1/index.html)

Ruby 语言常以其灵活性为人所称道.正如 Dick Sites 所言,您可以 "为了编程而编程".Ruby on Rails 扩展了核心 Ruby 语言,但正是 Ruby 本身使得这种扩展成为了可能.Ruby on Rails 使用了该语言的灵活性,这样一来,无需太多样板或额外的代码就可以轻松编写高度结构化的程序:无需额外工作,就可以获得大量标准的行为.虽然这种轻松自由的行为并不总是完美的,但毕竟您可以无需太多工作就可以获得很多好的架构.

例如,Ruby on Rails 基于模型-视图-控制器(Model-View-Controller,MVC)模式,这意味着大多数 Rails 应用程序都可以清晰地分成三个部分.模型部分包含了管理应用程序数据所需的行为.通常,在一个 Ruby on Rails 应用程序中,模型和数据库表之间的关系是 1:1；Ruby on Rails 默认使用的对象关系映射(ORM)ActiveRecord 负责管理模型与数据库的交互,这意味着 Ruby on Rails 程序通常都具有(如果有的话)很少量的 SQL 代码.第二个部分是视图,它包含创建发送至用户的输出所需要的代码；它通常由 HTML、JavaScript 等组成.最后的一个部分是控制器,它将来自用户的输入转变为正确的模型,然后使用适当的视图呈现响应.

Rails 的倡导者通常都乐于将其易用性方面的提高归功于 MVC 范型 — 以及 Ruby 和 Rails 二者的其他一些特性,并称很少有程序员能够在较短的时间内创建更多的功能.当然,这意味着投入到软件开发的成本将能够产生更多的商业价值,因此 Ruby on Rails 开发愈发流行.

不过,最初的开发成本并不是事情的全部,还有其他的后续成本需要考虑,比如应用程序运行的维护成本和硬件成本.Ruby on Rails 开发人员通常会使用测试和其他的敏捷开发技术来降低维护成本,但是这样一来,很容易忽视具有大量数据的 Rails 应用程序的有效运行.虽然 Rails 能够简化对数据库的访问,但它并不总是能够如此有效.

## Rails 应用程序为何运行缓慢?
Rails 应用程序之所以运行缓慢,其中有几个很基本的原因.第一个原因很简单:Rails 总是会做一些假设为您加速开发.通常,这种假设是正确而有帮助的.不过,它们并不总能有益于性能,并且还会导致资源使用的效率低下 — 尤其是数据库资源.

例如,使用等同于 `SELECT * `的一个 SQL 语句,ActiveRecord 会默认选择查询上的所有字段.在具有为数众多的列的情况下 — 尤其是当有些字段是巨大的 VARCHAR 或 BLOB 字段时 — 就内存使用和性能而言这种行为很有问题.

另一个显著的挑战是 **N+1** 问题,本文将对此进行详细的探讨.这会导致很多小查询的执行,而不是一个单一的大查询.例如,ActiveRecord 无从知道一组父记录中的哪一个会请求一个子记录,所以它会为每个父记录生成一个子记录查询.由于每查询的负荷,这种行为将导致明显的性能问题.

其他的挑战则更多地与  Ruby on Rails 开发人员的开发习惯和态度相关.由于 ActiveRecord 能够让如此众多的任务变得轻而易举,Rails 开发人员常常会形成 "SQL 不怎样" 的一种态度,即便在更适合使用 SQL 的时候,也会避免 SQL.创建和处理数量巨大的 ActiveRecord 对象的速度会非常缓慢,所以在有些情况下,直接编写一个无需实例化任何对象的 SQL 查询会更快些.

由于 Ruby on Rails 常被用来降低开发团队的规模,又由于 Ruby on Rails 开发人员通常都会执行部署和维护生产中的应用程序所需的一些系统管理任务,因此若对应用程序的环境知之甚少,就很可能出问题.操作系统和数据库有可能未被正确设置.比如,虽然并不最优,`MySQL my.cnf` 设置常常在 Ruby on Rails 部署内保留它们的默认设置.此外,可能还会缺少足够的监控和基准测试工具来提供性能的长期状况.当然,这并不是在责怪 Ruby on Rails 开发人员；这是非专业化导致的后果；在有些情况下,Rails 开发人员有可能是这两个领域的专家.

最后一个问题是 Ruby on Rails 鼓励开发人员在本地环境中进行开发.这么做有几个好处 — 比如,开发延迟的减少和分布性的提高 — 但它并不意味着您可以因为工作站规模的减少而只处理有限的数据集.他们如何开发以及代码将被部署于何处之间的差异可能会是一个大问题.即便您在一个性能良好的轻载本地服务器上处理小规模的数据已经很长一段时间,也会发现对于拥塞的服务器上的大型数据此应用程序会有很明显的性能问题.

当然,Rails 应用程序具有性能问题的原因可能有很多.查出 Rails 应用程序有何潜在性能问题的最佳方法是,利用能为您提供可重复、准确度量的诊断工具.

## 检测性能问题
最好的工具之一是 Rails 开发日志,它通常位于每个开发机器上的 `log/development.log` 文件内.它具有各种综合指标:响应请求所花费的总时间、花费在数据库内的时间所占的百分比、生成视图所花时间的百分比等.此外,还有一些工具可用来分析此日志文件,比如 development-log-analyzer.

在生产期间,通过查看 `mysql_slow_log`可以找到很多有价值的信息.更为全面的介绍超出了本文的讨论范围,更多信息可以在 [参考资料 ](http://www.ibm.com/developerworks/cn/opensource/os-Railsn1/index.html#resources)部分找到.

其中一个最强大也是最为有用的工具是 query_reviewer 插件(参见 [参考资料](http://www.ibm.com/developerworks/cn/opensource/os-Railsn1/index.html#resources)).这个插件可显示在页面上有多少查询在执行以及页面生成需要多长时间.并且它还会自动分析 ActiveRecord 生成的 SQL 代码以便发现潜在问题.例如,它能找到不使用 MySQL 索引的查询,所以如果您忘记了索引一个重要的列并由此造成了性能问题,那么您将能很容易地找到这个列(有关 MySQL 索引的更多信息,参见 [参考资料](http://www.ibm.com/developerworks/cn/opensource/os-Railsn1/index.html#resources)).此插件在一个弹出的 `<div>`(只在开发模式下可见)中显示了所有这类信息.

最后,不要忘记使用类似 Firebug、yslow、Ping 和 tracert 这样的工具来检测性能问题是来自于网络还是资源加载问题.

接下来,让我们来看具体的一些 Rails 性能问题及其解决方案.

## N+1 查询问题
N+1 查询问题是 Rails 应用程序最大的问题之一.例如,清单 1 内的代码能生成多少查询?此代码是一个简单的循环,遍历了一个假想的 post 表内的所有 post,并显示 post 的类别和它的主体.
### 清单 1. 未优化的 Post.all 代码
{% highlight ruby %}
<%@posts = Post.all(@posts).each do |p|%>
  <h1><%=p.category.name%></h1>
  <p><%=p.body%></p>
<%end%>
{% endhighlight %}

答案:上述代码生成了一个查询外加  `@posts` 内的每行一个查询.由于每查询的负荷,这可能会成为一个很大的挑战.罪魁祸首是对 `p.category.name` 的调用.这个调用只应用于该特定的 `post` 对象,而不是整个 `@posts` 数组.幸好,通过使用立即加载,我们可以修复这个问题.

立即加载 意味着 Rails 将自动执行所需的查询来加载任何特定子对象的对象.Rails 将使用一个 `JOIN` SQL 语句或一个执行多个查询的策略.不过,假设指定了将要使用的所有子对象,那么将永远不会导致 N+1 的情形,在 N+1 情形下,一个循环的每个迭代都会生成额外的一个查询.清单 2 是对 清单 1 内代码的修订,它使用了立即加载来避免 N+1 问题.

### 清单 2. 用立即加载优化后的 Post.all 代码
{% highlight ruby %}
<%@posts = Post.find(:all, :include=>[:category]
  @posts.each do |p|%>
  <h1><%=p.category.name%></h1>
  <p><%=p.body%></p>
<%end%>
{% endhighlight %}

该代码最多生成两个查询,而不管在此 `posts` 表内有多少行.

当然,并不是所有情况都如此简单.处理复杂的 N+1 查询情况需要更多的工作.那么做这么多努力值得么?让我们来做一些快速的测试.

## 测试 N+1
使用清单 3 内的脚本,可以发现查询可以达到 — 多慢 — 或多快. 清单 3 展示了如何在一个独立脚本中使用 ActiveRecord 来建立一个数据库连接、定义表并加载数据.然后,可以使用 Ruby 的内置基准测试库来查看哪种方式更快,快多少.

### 清单 3. 立即加载基准测试脚本
{% highlight ruby %}
require 'Rubygems'
require 'faker'
require 'active_record'
require 'benchmark'

# This call creates a connection to our database.

ActiveRecord::Base.establish_connection(
  :adapter  => "mysql",
  :host     => "127.0.0.1",
  :username => "root", # Note that while this is the default setting for MySQL,
  :password => "",     # a properly secured system will have a different MySQL
                            # username and password, and if so, you'll need to
                            # change these settings.
  :database => "test")

# First, set up our database...
class Category <  ActiveRecord::Base
end

unless Category.table_exists?
  ActiveRecord::Schema.define do
    create_table :categories do |t|
        t.column :name, :string
    end
  end
end

Category.create(:name=>'Sara Campbell\'s Stuff')
Category.create(:name=>'Jake Moran\'s Possessions')
Category.create(:name=>'Josh\'s Items')
number_of_categories = Category.count

class Item <  ActiveRecord::Base
  belongs_to :category
end

# If the table doesn't exist, we'll create it.

unless Item.table_exists?
  ActiveRecord::Schema.define do
    create_table :items do |t|
        t.column :name, :string
        t.column :category_id, :integer
    end
  end
end

puts "Loading data..."

item_count = Item.count
item_table_size = 10000

if item_count < item_table_size
  (item_table_size - item_count).times do
    Item.create!(:name=>Faker.name,
                 :category_id=>(1+rand(number_of_categories.to_i)))
  end
end

puts "Running tests..."

Benchmark.bm do |x|
  [100,1000,10000].each do |size|
    x.report "size:#{size}, with n+1 problem" do
      @items=Item.find(:all, :limit=>size)
      @items.each do |i|
        i.category
      end
    end
    x.report "size:#{size}, with :include" do
      @items=Item.find(:all, :include=>:category, :limit=>size)
      @items.each do |i|
        i.category
      end
    end
  end
end
{% endhighlight %}

这个脚本使用 `:include` 子句测试在有和没有立即加载的情况下对 100、1,000 和 10,000 个对象进行循环操作的速度如何.为了运行此脚本,您可能需要用适合于您的本地环境的参数替换此脚本顶部的这些数据库连接参数.此外,需要创建一个名为 test 的 MySQL 数据库.最后,您还需要 ActiveRecord 和 faker 这两个 gem,二者可通过运行 `gem install activerecord faker` 获得.

在我的机器上运行此脚本生成的结果如清单 4 所示.

### 清单 4. 立即加载的基准测试脚本输出
{% highlight ruby %}
-- create_table(:categories)
   -> 0.1327s
-- create_table(:items)
   -> 0.1215s
Loading data...
Running tests...
      user     system      total        real
size:100, with n+1 problem  0.030000   0.000000   0.030000 (  0.045996)
size:100, with :include  0.010000   0.000000   0.010000 (  0.009164)
size:1000, with n+1 problem  0.260000   0.040000   0.300000 (  0.346721)
size:1000, with :include  0.060000   0.010000   0.070000 (  0.076739)
size:10000, with n+1 problem  3.110000   0.380000   3.490000 (  3.935518)
size:10000, with :include  0.470000   0.080000   0.550000 (  0.573861)
{% endhighlight %}

在所有情况下,使用 :include 的测试总是更为迅速 — 分别快 5.02、4.52 和 6.86 倍.当然,具体的输出取决于您的特定情况,但立即加载可明显导致显著的性能改善.

## 嵌套的立即加载
如果您想要引用一个嵌套的关系 — 关系的关系,又该如何呢? 清单 5 展示了这样一个常见的情形:循环遍历所有的 post 并显示作者的图像,其中 Author 与 Image 是 belongs_to 的关系.

### 清单 5. 嵌套的立即加载用例
{% highlight ruby %}
@posts = Post.all
@posts.each do |p|
  <h1><%=p.category.name%></h1>
  <%=image_tag p.author.image.public_filename %>
  <p><%=p.body%>
 <%end%>
 {% endhighlight %}

此代码与之前一样亦遭遇了相同的 N+1 问题,但修复的语法却没有那么明显,因为这里所使用的是关系的关系.那么如何才能立即加载嵌套关系呢?

正确的答案是使用 `:include` 子句的哈希语法.清单 6 给出了使用哈希语法的一个嵌套的立即加载.

### 清单 6. 嵌套的立即加载解决方案
{% highlight ruby %}
@posts = Post.find(:all, :include=>{ :category=>[],
                                       :author=>{ :image=>[]}} )
@posts.each do |p|
  <h1><%=p.category.name%></h1>
  <%=image_tag p.author.image.public_filename %>
  <p><%=p.body%>
 <%end%>
 {% endhighlight %}

正如您所见,您可以嵌套哈希和数组实量(literal).请注意在本例中哈希和数组之间的惟一区别是哈希可以含有嵌套的子条目,而数组则不能.否则,二者是等效的.


## 间接的立即加载
并非所有的 N+1 问题都能很容易地察觉到.例如,清单 7 能生成多少查询?
### 清单 7. 间接的立即加载示例用例
{% highlight ruby %}
  <%@user = User.find(5)
    @user.posts.each do |p|%>
      <%=render :partial=>'posts/summary',  :locals=>:post=>p
     %> <%end%>
     {% endhighlight %}

当然,决定查询的数量需要对 `posts/summary` partial 有所了解.清单 8 中显示了这个 partial.

### 清单 8. 间接立即加载 partial: posts/_summary.html.erb
  `<h1><%=post.user.name%></h1>`
不幸的是,答案是 **清单 7*** 和 **清单 8*** 在 post 内每行生成一个额外查询,查找用户的名字 — 即便 post 对象由 ActiveRecord 从一个已在内存中的 User 对象自动生成.简言之,Rails 并不能关联子记录与其父记录.
修复方法是使用自引用的立即加载.基本上,由于 Rails 重载由父记录生成的子记录,所以需要立即加载这些父记录,就如同父与子记录之间是完全分开的关系一样.代码如清单 9 所示.

### 清单 9. 间接的立即加载解决方案
  {% highlight ruby %}
  <%@user = User.find(5, :include=>{:posts=>[:user]})
  ...snip...
  {% endhighlight %}

虽然有悖于直觉,但这种技术与上述技术的工作原理大致相似.但是,很容易使用这种技术进行过多的嵌套,尤其是在体系结构复杂的情况下.简单的用例还好,比如 清单 9 内所示的,但繁复的嵌套也会出问题.在一些情况下,过多地加载 Ruby 对象有可能会比处理 N+1 问题还要缓慢 — 尤其是当每个对象并没有被整个树遍历时.在该种情况下,N+1 问题的其他解决方案可能更为适合.

一种方式是使用缓存技术.Rails V2.1 内置了简单的缓存访问.使用 Rails.cache.read、 Rails.cache.write 及相关方法,可以轻松创建自己的简单缓存机制,并且后端可以是一个简单的内存后端、一个基于文件的后端或一个分布式缓存服务器.在 [参考资料](http://www.ibm.com/developerworks/cn/opensource/os-Railsn1/index.html#resources) 部分可以找到有关 Rails 内置缓存支持的更多信息.但您无需创建自己的缓存解决方案；您可以使用一个预置的 Rails 插件,比如 Nick Kallen 的 cache money 插件.这个插件提供了 write-through 缓存并以 Twitter 上使用的代码为基础.更多信息参见 [参考资料](http://www.ibm.com/developerworks/cn/opensource/os-Railsn1/index.html#resources).

当然,并不是所有的 Rails 问题都与查询的数量有关.

## Rails 分组和聚合计算
您可能遇到的一个问题是在 Ruby 所做的工作本应由数据库完成.这考验了 Ruby 的强大程度.很难想象在没有任何重大激励的情况下人们会自愿在 C 中重新实现其数据库代码的各个部分,但很容易在 Rails 内对 ActiveRecord 对象组进行类似的计算.但是,Ruby 总是要比数据库代码慢.所以请不要使用纯 Ruby 的方式执行计算,如清单 10 所示.

### 清单 10. 执行分组计算的不正确方式
{% highlight ruby %}
 all_ages = Person.find(:all).group_by(&:age).keys.uniq
  oldest_age = Person.find(:all).max
  {% endhighlight %}

相反,Rails 提供了一系列的分组和聚合函数.可以像清单 11 所示的那样使用它们.

### 清单 11. 执行分组计算的正确方式
{% highlight ruby %}
 all_ages = Person.find(:all, :group=>[:age])
  oldest_age = Person.calcuate(:max, :age)
  {% endhighlight %}
ActiveRecord::Base#find 有大量选项可用于模仿 SQL.更多信息可以在 Rails 文档内找到.注意,calculate 方法可适用于受数据库支持的任何有效的聚合函数,比如 `:min`、`:sum` 和 `:avg`.此外,calculate 能够接受若干实参,比如 `:conditions`.查阅 Rails 文档以获得更详细的信息.

不过,并不是在 SQL 内能做的所有事情在 Rails 内也能做.如果插件不够,可以使用定制 SQL.

## 用 Rails 定制 SQL
假设有这样一个表,内含人的职业、年龄以及在过去一年中涉及到他们的事故的数量.可以使用一个定制  SQL 语句来检索此信息,如清单 12 所示.

### 清单 12. 用 ActiveRecord 定制 SQL 的例子
{% highlight ruby %}
sql = "SELECT profession,
              AVG(age) as average_age,
              AVG(accident_count)
         FROM persons
        GROUP
           BY profession"

Person.find_by_sql(sql).each do |row|
  puts "#{row.profession}, " <<
       "avg. age: #{row.average_age}, " <<
       "avg. accidents: #{row.average_accident_count}"
end
{% endhighlight %}

这个脚本应该能生成清单 13 所示的结果.

### 清单 13. 用 ActiveRecord 定制 SQL 的输出
{% highlight ruby %}
  Programmer, avg. age: 18.010, avg. accidents: 9
  System Administrator, avg. age: 22.720, avg. accidents: 8
  {% endhighlight %}
当然,这是最简单的例子.您可以自己想象一下如何能将此例中的 SQL 扩展成一个有些复杂性的 SQL 语句.您还可以使用 `ActiveRecord::Base.connection.execute` 方法运行其他类型的 SQL 语句,比如 ALTER TABLE 语句,如清单 14 所示.

### 清单 14. 用 ActiveRecord 定制非查找型 SQL
{% highlight ruby %}
  ActiveRecord::Base.connection.execute "ALTER TABLE some_table CHANGE COLUMN..."
  {% endhighlight %}

大多数的模式操作,比如添加和删除列,都可以使用 Rails 的内置方法完成.但如果需要,也可以使用执行任意 SQL 代码的能力.

##结束语

与所有的框架一样,如果不多加小心和注意,Ruby on Rails 也会遭遇性能问题.所幸的是,监控和修复这些问题的技术相对简单且易学,而且即便是复杂的问题,只要有耐心并对性能问题的源头有所了解,也是可以解决的.
