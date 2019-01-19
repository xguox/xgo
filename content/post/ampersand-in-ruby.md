---
title: "Ruby 中的 &"
date: 2014-06-20T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

才发现, `&` 还是一个按位与运算符, 好像应该用才想起, 貌似以前看 'Ruby 编程语言' 内本书的时候有看到过, 不过感觉比较少用到, 没怎么去记住.

今个儿[看到](http://ablogaboutcode.com/2012/01/04/the-ampersand-operator-in-ruby/)

```
14 & 13

```

输出的是 12, 才发现又得补课了.


```
"#{14.to_s(2)} & #{13.to_s(2)} = #{12.to_s(2)}"

```


数组取共有元素的用法是知道的, 但是, 对于`FalseClass` `NilClass` 还有 `TrueClass` 来说, `&` 相当于 'AND', 这个还真不知道 = .  =

```
irb(main):001:0> false & true
=> false
irb(main):002:0> nil & true
=> false
irb(main):003:0> true & Object.new
=> true
irb(main):004:0> Object.new & true
=> NoMethodError: undefined method `&' for #<Object:0x007f9e7ac96420>

```


转换 Block 和 Proc 的用法也是没问题的.

- if object is a block, it converts the block into a simple proc.
- if object is a Proc, it converts the object into a block while preserving the lambda? status of the object.
- if object is not a Proc, it first calls #to_proc on the object and then converts it into a block.


---------------------

__update:__

前几天看到这么个写法: `params[resource] &&= send(method)`

赶脚好奇葩的样子, 长得像`+=` 这类运算, 但是如果按这个理解的话不太妥 = .  =

查之发现,

不单是有 `&&=`, 还有 `||=`

比如 `options[:name] ||= 'Bond'` 相当于 `options[:name] = 'Bond' unless options[:name]`

而 `options[:name] &&= options[:name].strip` 相当于 `options[:name] = options[:name].strip if options[:name]`
