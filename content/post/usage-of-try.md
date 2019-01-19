---
title: "Rails中try的用法"
date: 2012-04-20T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

居然今晚才发现Rails中try的用法,初一看,怎么那么像印象中某些语言的异常处理?不对啊,Ruby中的异常处理不是长这个样子的.
查了下Rails的API才了解了大致用法.

API:

不使用try的时候是

```ruby
@person && @person.name
```

需确保@person不是nil,否则会报错

```ruby
NoMethodError: undefined method `name' for nil:NilClass
```

如果用try的话

```ruby
@person.try(:name)
```

如果@person为nil则返回nil而不是报错

另外,try还可以接收block作为参数,

```ruby
Person.try(:find, 1)
@people.try(:collect) {|p| p.name}
```

漏了一句,API开头就说了,像常规的Ruby的Object#send 那样工作
看了下源码,貌似是利用了__send__实现的.最近在研究Ruby的元编程.