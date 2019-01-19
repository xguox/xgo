---
title: "重拾Ruby (I)"
date: 2013-11-04T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

真心尴尬,学习并使用了一年多的 Ruby/Rails,然后些个月不碰就忘了大半了.原本就不高端,这一回滚跟重头再来差不多了. 语法还能快速过了一遍.(主要通过 [CodeSchool](https://www.codeschool.com/)) 元编程,DSL那一块接下来可要费点劲了,毕竟原先也没完全掌握到.

今天学到两个之前没注意的 tricks.

一个是关于`to_s`的,另外个是关于 `Module` .


##### to_s


```ruby
class Game
  attr_accessor :name, :year, :system
  attr_reader :created_at
  def initialize(name, options={})
    self.name = name
    self.year = options[:year]
    self.system = options[:system]
    @created_at = Time.now
  end

  def to_s
    self.name
  end

  def description
    "#{self} was released in #{self.year}."
  end
end

class ConsoleGame < Game
  def to_s
    "#{self.name} - #{self.system}"
  end
end

```

其中的字符串插值`#{self}`其实相当于隐式调用了`to_s`,在这里也就是`self.to_s` 好吧, 我傻逼了,字符串插值当然是不管啥都执行`to_s`. 不过我想说的不是这个.而是牵扯到`puts`和`p`




##### puts and p and inspect

调用一些例子:


```ruby
class A
  def initialize(string, number)
    @string = string
    @number = number
  end

  def to_s
    "In to_s:\n   #{@string}, #{@number}\n"
  end
  def to_a
    "In to_a:\n   #{@string}, #{@number}\n"
  end
end
puts a = A.new("hello world", 5)

```

输出

```ruby
In to_s:
   hello world, 5

```

这里的`to_s`在对象被实例化之后隐式调用了

然后是

```ruby
class Item
  def inspect
    "Result of inspect"
  end
end

puts Item.new
puts Item.new.to_s
p Item.new


```

输出


```ruby
#<Item:0x8f85568>
#<Item:0x8f8552c>
Result of inspect

```

`puts`通常输出的是 对象调用`to_s`的结果,而`p`则是直接输出`inspect`


##### inspect vs to_s


另一个例子

```ruby
class Item
  def initialize(item_name, qty)
    @item_name = item_name
    @qty = qty
  end
end

item = Item.new("a",1)

puts item
p item


```

输出

```ruby
#<Item:0x8f85388>
#<Item:0x8f85388 @item_name="a", @qty=1>

```

可以看到, `puts`打印出类名然后跟着一个十六进制的数,这个数字跟该对象在内存中的存储位置相关,不过我们很少会使用到.

而`p`的话是打印出类名及该对象的所有的实例变量,这个在debug的时候非常有用.

上面这个例子说明了`p`和`puts`的不同,但是,有时候你会想要自定义这些方法的输出形式. 这个可以通过重写`to_s`来完成.

如下面的例子


```ruby
class Item
  def initialize(item_name, qty)
    @item_name = item_name
    @qty = qty
  end

  def to_s
    "#{@item_name}: #{@qty}"
  end
end

item = Item.new("a",1)

puts item
p item


```

输出


```ruby
a: 1
a: 1

```


此时,`p`和`puts`是相同的,因为`to_s`被重写,Ruby会把它当作默认的`inspect`结果.除非你重写`inspect`方法. 如前边的例子


参考[RubyMonk](http://Rubymonk.com/learning/books/4-Ruby-primer-ascent/chapters/45-more-classes/lessons/108-displaying-objects)

[http://stackoverflow.com/questions/12040527/Ruby-automatically-calls-to-s-method-when-object-is-created/19751192#19751192](http://stackoverflow.com/questions/12040527/Ruby-automatically-calls-to-s-method-when-object-is-created/19751192#19751192)



##### Module

一般的,把 module 的方法添加到类中有两种方式(用途也不同).

一种是使用 include 添加后作为实例方法; 另一种是使用 extend ,对应的是作为类方法使.

当然,这是比较多用到的, 再还有一种则是扩展单个对象.
直接 copy 官方文档的例子

```ruby
module Mod
  def hello
    "Hello from Mod.\n"
  end
end

class Klass
  def hello
    "Hello from Klass.\n"
  end
end

k = Klass.new
k.hello         #=> "Hello from Klass.\n"
k.extend(Mod)   #=> #<Klass:0x401b3bc8>
k.hello         #=> "Hello from Mod.\n"

```


当需要作为类方法和实例方法添加到类当中的时候, 当然, 可以同时使用`include`和`extend`, 不过还有另一种简便一些的方法 **Hooks - self.included**

比如

```ruby
module LibraryUtils
  def self.included(base)
    base.extend(ClassMethods)
  end

  def add_game(game)
  end

  def remove_game(game)
  end

  module ClassMethods
    def search_by_game_name(name)
    end
  end
end

class AtariLibrary
  include LibraryUtils
end

# 当前 module 被 base 包含
# module名 ClassMethods 可以任意取

```


最后, 还有可以通过调用`ActiveSupport::Concern`.参考 [Rails](http://api.RubyonRails.org/) 文档
典型的 module 是:

```ruby
module M
  def self.included(base)
    base.extend ClassMethods
    base.class_eval do
      scope :disabled, -> { where(disabled: true) }
    end
  end

  module ClassMethods
    ...
  end
end

```


使用 `ActiveSupport::Concern` 上边的例子可以这么改写:


```ruby
require 'active_support/concern'

module M
  extend ActiveSupport::Concern

  included do
    scope :disabled, -> { where(disabled: true) }
  end

  module ClassMethods
    ...
  end
end

```

此外, 它可以优雅地处理 module 之间的依赖. 如下例子,假定 module `Bar` 依赖于 module `Foo`. 通常的我们会这么写

```ruby
module Foo
  def self.included(base)
    base.class_eval do
      def self.method_injected_by_foo
        ...
      end
    end
  end
end

module Bar
  def self.included(base)
    base.method_injected_by_foo
  end
end

class Host
  include Foo # We need to include this dependency for Bar
  include Bar # Bar is the module that Host really needs
end

```

但是,`Host`为嘛要关心`Bar` 的依赖 `Foo`呢?为啥不直接在`Bar`里头引入`Foo`呢?

```ruby
module Bar
  include Foo
  def self.included(base)
    base.method_injected_by_foo
  end
end

class Host
  include Bar
end

```

不幸的是,这不起作用. 因为当`Foo`在`Bar`中included 的时候,`Foo` 中的`base`实际上是`Bar` module,而不是 `Host`类.这时候用`ActiveSupport::Concern`的话就可以杠杠地解决这个依赖问题.


```ruby
require 'active_support/concern'

module Foo
  extend ActiveSupport::Concern
  included do
    def self.method_injected_by_foo
      ...
    end
  end
end

module Bar
  extend ActiveSupport::Concern
  include Foo

  included do
    self.method_injected_by_foo
  end
end

class Host
  include Bar # works, Bar takes care now of its dependencies
end

```


最后,附上`included`源码

```ruby
# File activesupport/lib/active_support/concern.rb, line 118
def included(base = nil, &block)
  if base.nil?
    @_included_block = block
  else
    super
  end
end

```










