---
title: "语法补习: Ruby 方法 source_location 和奇葩的逗号(,)"
date: 2016-10-16T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

```ruby
User.new.method(:order_commissions).source_location
# => ["....../vendor/bundle/gems/activerecord-4.2.6/lib/active_record/associations/builder/association.rb", 114]

User.method(:generate_token).source_location
# ["....../app/models/user.rb", 50]
```

这真的没有语法错误 = . =

```ruby
v1, = [["a", "b", "c"], "d", "e"]
v2, = v1
[33] pry(main)> v2
=> "a"
```