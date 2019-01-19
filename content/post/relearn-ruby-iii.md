---
title: "重拾Ruby (III)"
date: 2013-11-07T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

不把这些当做笔记卸写下来总隐隐约约觉得自己没掌握, 所以还是继续做笔记.

### Proc

lambda 在1.9之后的写法(之前的也可以用)

```ruby
lamb = -> { }
```


proc 对象 和 block 对象之间的转换.

#### 调用方法的时候参数前加 &


```ruby
tweets.each(&printer)
 # 把 proc 对象 转换成 block 对象 each 后面的是 block ,不是传参数
```


#### 定义方法的时候参数前加 &

```ruby
def each(&block)
 # 把 block 对象 转换成 proc 对象, 把 block 转换成 proc 才能作为参数

```


PS. method 对象转换成 block 对象

#### symbol

`tweets.map { |tweet| tweet.user }`

Same as

`tweets.map(&:user)`

#### block_given?

#### closure

```ruby

def tweet_as(user)
  lambda { |tweet| puts "#{user}: #{tweet}" }
end
```

当 lambda 被创建后局部变量(在这即user) 被保存起来

```ruby
gregg_tweet = tweet_as("greggpollack")

 # 相当于创建了   lambda { |tweet| puts "greggpollack: #{tweet}" }

gregg_tweet.call("Test!")

 # greggpollack: Test!
```



## self

```ruby
puts "Outside the class : #{self}"

 # Outside the class : main

class Tweet
  puts "Inside the class : #{Tweet}"
end

 # Inside the class : Tweet
```


### class_eval sets `self` to the given class and executes the block

### instance_eval sets `self` to the given instance and executes the block








