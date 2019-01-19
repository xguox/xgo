---
title: "Rails autoload_paths & eager_load_paths"
date: 2016-07-19T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

## autoload_paths

假设 Rails 项目根目录下有如下目录以及 .rb 文件,

`(root/)extras/foo.rb`

如果啥也不干, 直接打开 rails console:

```ruby
[1] pry(main)> defined?(Foo)
=> nil

[2] pry(main)> Foo
NameError: uninitialized constant Foo
from (pry):2:in `<main>'

```

在 application.rb 中加入一行:

`config.autoload_paths += %W(#{config.root}/extras)`


再重启 rails console:

```ruby
[1] pry(main)> defined?(Foo)
=> nil

[2] pry(main)> Foo
=> Foo

[3] pry(main)> defined?(Foo)
=> "constant"
```

这是 autoload_paths 延迟加载的作用, 只有当用到的时候再去查找这个常量. 而开发环境 app 目录下的所有文件都是这样延迟加载的.

**但是, 在生产环境中:**

```ruby
# app/models/segment.rb  已经预加载
[1] pry(main)> defined?(Segment)
=> "constant"

# foo.rb 还是延迟加载
[2] pry(main)> defined?(Foo)
=> nil

```

## eager_load_paths

这个时候需要的是 eager_load_paths

在 application.rb 中加上

`config.eager_load_paths += %W( #{config.root}/extras )`


```ruby
# extras/foo.rb 已经预加载
[1] pry(main)> defined?(Foo)
=> "constant"
```

