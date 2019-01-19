---
title: "Rails render collection 的魔法"
date: 2017-08-17T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

都知道的, 在 Rails 的 View 里边渲染集合的时候, 会用到 render 方法参数的 **collection** 选项

```ruby

<%= render partial: "product", collection: @products %>

```

而不是遍历集合来渲染单个模板.

渲染集合还有个简写形式. 假设 `@products` 是 `product` 实例集合, 在 `index.html.erb `中可以直接写成下面的形式, 得到的结果是一样的:

```ruby

<%= render @products %>

```

这里,  Rails 做的魔法其实是去找遍历成员的 **to_partial_path**

**action_view/renderer/partial_renderer (Rails 4.2)**

```ruby
def partial_path(object = @object)
  object = object.to_model if object.respond_to?(:to_model)

  path = if object.respond_to?(:to_partial_path)
    object.to_partial_path
  else
    raise ArgumentError.new("'#{object.inspect}' is not an ActiveModel-compatible object. It must implement :to_partial_path.")
  end

  if @view.prefix_partial_path_with_controller_namespace
    prefixed_partial_names[path] ||= merge_prefix_into_object_path(@context_prefix, path.dup)
  else
    path
  end
end
```

打开 rails console 可以试试

```ruby
[1] pry(main)> User.new.to_partial_path
=> "users/user"
```

这里也可以把 user 这个 model 的 **to_partial_path** 重写,  返回表示渲染路径的字符串,

如果你的某个 **PORO** 实现了 **to_partial_path**, 那对应的 collection 也可以直接用类似的方式去 render