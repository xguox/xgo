---
title: "语法补习: Refinements in Ruby 2.1"
date: 2016-07-16T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

Ruby 可以随时打开一个类进行 Monkey Patch, 但是这是一个比较危险的动作, 很容易引发一些意外, 而 2.0 开始, 加入的 **refinements** 则是为了让这一动作变得相对安全一些.

举个只为理解毫无实际意义的栗子:

```ruby
class String
  def simple_camelize
    split(/_/).map(&:capitalize).join
  end
end

module Whatever
  CONST_A = "active_record".simple_camelize
end

Whatever::CONST_A
=> "ActiveRecord"
```

-----------

```ruby
class String
  def simple_camelize
    "just simple_camelize"
  end
end

module StringRefinements
  refine String do
    def simple_camelize
      split(/_/).map(&:capitalize).join
    end
  end
end

module Whatever
  using StringRefinements
  CONST_A = "active_record".simple_camelize
end

Whatever::CONST_A
=> "ActiveRecord"

"active_support".simple_camelize
=> "just simple_camelize"

# 咳~~咳, 再次打开 Whatever

module Whatever
  CONST_B = "active_job".simple_camelize
end
Whatever::CONST_B
=> "just simple_camelize"

class A
  include Whatever

  def test
    puts "action_pack".simple_camelize
    # 输出 just simple_camelize
  end
end

class B
  using StringRefinements

  def test
    puts "action_pack".simple_camelize  # 输出 ActionPack
  end
end

```
