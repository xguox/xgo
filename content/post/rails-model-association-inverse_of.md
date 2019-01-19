---
title: "[Rails] Model 关联选项之 :inverse_of"
date: 2015-10-17T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

在小项目没怎么遇见过的 model 之间的 **has_many** 关联选项 `:inverse_of`, 用来指定相对应的 `belongs_to` 关联的名字, (反之对于 belongs_to 也一样,文邹邹的 (・_・ヾ ). 不能与 `:through` , `:as`, `:polymorphic` 一起使用.

```ruby
class Product < ActiveRecord::Base
  has_many :request_items, :inverse_of => :product,
           :class_name => 'PurchaseRequestItem'
end

class PurchaseRequestItem < ActiveRecord::Base
  belongs_to :purchase_request, :inverse_of => :request_items,
             :class_name => "PurchaseRequest"
  belongs_to :product, :inverse_of => :request_items,
             :class_name => "Product"
end
```

```ruby
[1] pry(main)> purchase_request_item = PurchaseRequestItem.last
  PurchaseRequestItem Load (0.2ms)  SELECT `purchase_request_items`.* FROM
  `purchase_request_items` ORDER BY `purchase_request_items`.`id` DESC LIMIT 1
=> #<PurchaseRequestItem id: 56387, purchase_request_id: 2979, product_id: 311795>
[2] pry(main)> product = Product.find 311795
  Product Load (0.5ms)  SELECT `products`.* FROM `products` WHERE `products`.`id` = 311795
  AND (`products`.deleted_at IS NULL) LIMIT 1
=> #<Product id: 311795, status: 1, is_popular: false>
```

```ruby
# 没有加上 :inverse_of 选项的话是酱紫的

[3] pry(main)> purchase_order_item.product == product
  Product Load (0.4ms)  SELECT `products`.* FROM `products` WHERE `products`.`id` = 311795
  AND (`products`.deleted_at IS NULL) LIMIT 1
=> true
[4] pry(main)> product.status=3
=> 3
[5] pry(main)> purchase_order_item.product.status==product.status
=> false
```

```ruby
# 加上 :inverse_of 是酱紫的

[3] pry(main)> purchase_order_item.product == product
=> true
[4] pry(main)> product.status=3
=> 3
[5] pry(main)> pri.product.status==product.status
=> true
```

如果没有 **:inverse_of** 的话, 会在数据库中查找多一次, 而如果有 **:inverse_of** 的话, 是直接从内存里面取的.

好吧, **Rails 4.1** 开始默认是加上了 `:inverse_of` 的. 只是公司有项目还跑在 Rails 3, 所以, 还是得手工加上这个 选项.
