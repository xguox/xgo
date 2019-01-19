---
title: "重拾Ruby (II)"
date: 2013-11-05T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

继续复习 + 笔记, 今天要写的是 Dynamic Classes & Methods


##### Struct

一般的, 定义一个类的做法如下：

```ruby
class Game
  attr_accessor :name, :year, :system

  def initialize(name, year, system)
    self.name = name
    self.year = year
    self.system = system
  end
end
```


而鉴于以上这个例子的数据结构比较简单, 所以其实我们可以使用Struct来写之:

```ruby
Game = Struct.new(:name, :year, :system)

```

如果需要添加实例方法则可以这么干:

```ruby
Game = Struct.new(:name, :year, :system) do
  def to_s
    "#{name},#{year},#{system}"
  end
end
```


如果需要定义的 data 比 behavior 要多的话, 推荐使用 Struct 来创建类, 反之则使用传统的方法.

##### alias_method

```ruby
class Timeline

  def initialize(tweets=[])
   @tweets = tweets
  end

  def tweets
    @tweets
  end

  def contents
    @tweets
  end

end
```

由于 tweets 和 contents 两个方法体其实是一样的只是方法名不同, 为免重复我们可以这么干

```ruby
class Timeline
  def initialize(tweets=\[\])
   @tweets = tweets
  end

  def tweets
    @tweets
  end

 #实际上其实这是getter方法,所以其实这里可以这么写

 # attr_reader :tweets

  alias_method :contents, :tweets

end
```


更多例子, 假设如下在 Timeline 类

```ruby
class Timeline

  attr_accessor: :tweets

  def print
    puts tweets.join('\n')
  end
end

```


我们需要给 print 方法添加 authentication .如果由于某些原因我们不想改动现有的方法的话, 可以重新打开`Timeline` 这个类,然后使用 `alias_method`

```ruby
class Timeline

  alias_method :old_print, :print

  def print
    authenticate!
    old_print
  end

  #实际上就是旧有的 print 方法改名为 old_print, 而重写了 print 给它加上了 authenticate!
  #然后调用旧有方法 old_print

  def authenticate!
    # do sth authentication
  end
end
```


但是,需要注意的是,重新打开一个类是个危险的做法. 所以, 另一种做法是使用继承.

```ruby
class AuthenticatedTimeline
  def print
    authenticate!
    super
  end

  def authenticate!
    # do sth authentication
  end
end
```


好吧, 尴尬了,貌似 alias_method 没看到什么更实际的意义了 =.=

##### define_method

下边例子, 可以看到比较多的重复代码,

```ruby
class Tweet
  def draft
    @status = :draft
  end

  def posted
    @status = :posted
  end

  def deleted
    @status = : deleted
  end
```


使用 define_method 可以杠杠的减少这些重复. 并且,当需要添加新的 state 时候只需添加到 states 数组中即可.

```ruby
class Tweet
  states = [:draft, :posted, :deleted]

  states.each do |status|
    define_method status do
      @status = status
    end
  end
end
```




##### send

```ruby
class Timeline
  def initialize(tweets)
    @tweets = tweets
  end

  def contents
    @tweets
  end

  private
  def direct_messages
  end
end

tweets = %w{ 'Compiling!' 'Bundling...'}
timeline = Timeline.new(tweets)
```


`timeline.contents`
一般的, 我们是这么调用 contents 方法.

除此外,我们还可以使用 send

`timeline.send(:contents)`

等同于

`timeline.send("contents")`

另外,我们还可以用 send 来调用 private  或者 protected
`timeline.direct_message`

如果不希望调用 private 和 protected 的方法则可以用 `public_send`

尴尬,更具体用途有待挖掘.

##### method 方法

```ruby
class Timeline
  def initialize(tweets)
    @tweets = tweets
  end

  def contents
    @tweets
  end

  def show_tweet(index)
    puts @tweets[index]
  end
end

tweets = ['Compling!', 'Bundling...']
timeline = Timeline.new(tweets)

content_method = timeline.method(:contents)
 # => #<Method: Timeline#contents>

content_method.call
 # => ['Compling!', 'Bundling...']

show_method = timeline.method(:show_tweet)
 # => #<Method: Timeline#show_tweet>

show_method.call(0)
 # => Compiling!

(0..1).each(&show_method)
 # =>
 # Compiling!
 # Bundling...

 # 通过 & 把 method 对象转换成 block
 # same as
 # show_method.call(0)
 # show_method.call(1)

```

在 Ruby 中, 任何东西都是 object, 任意的一个方法同样,也是一个 object


#### Practice
重构

```ruby
class Library
  attr_accessor :games

  def each(&block)
    games.each(&block)
  end

  def map(&block)
    games.map(&block)
  end

  def select(&block)
    games.select(&block)
  end
end

```


好吧,没掌握熟练,各种转晕了

