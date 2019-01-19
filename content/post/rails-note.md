---
title: "Rails 小记两则"
date: 2012-05-27T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

最近在做个课程设计,好吧,俺老师是要求用JSP的,可惜我那弱爆了的水平是不可能完成的.于是顶着各种压力自己用Rails来做.

今天才发现个问题,原来mysql默认存储的int貌似最多是10位就爆了,原本我的数据库中有学号那个字段的并且migration是存为integer,结果录入数据时候就各种甭了,因为学校学号都是10位数,不管我输入的是什么,进了数据库的学号字段都变成 2147483648, 刚开始看日志,看程序都木有发现什么问题,纠结不能,查看数据库发现,每条记录的学号西端都是统一的,搜索一通才发现问题所在.于是直接用mysql工具将学号字段改成了BIGINT,杠杠的就啥事都没了

另外个是,用了will_paginate这个gem做分页,不过默认显示的是"previous"和"next",改为中文的"前一页"和"后一页"需要先在config/initializers/ 下新建一个will_paginate.rb,  and then在里头加入

```ruby
# coding: utf-8
WillPaginate::ViewHelpers.pagination_options[:previous_label ] =  "前一页"
WillPaginate::ViewHelpers.pagination_options[:next_label ] =  "后一页"
```
