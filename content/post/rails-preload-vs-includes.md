---
title: "[Rails] preload vs includes"
date: 2019-02-15T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

写了这么长时间的 **Rails**, 大部分时候都是用 **includes**, 鲜少使用到 **preload**. 所以, 没太在意这俩的区别. 最近写一个比较复杂的报表 SQL 时候总算踩到坑了.

# preload

会生成分开的两条查询, 类似:

```ruby
Product.preload(:taxon).load
# Product Load (33.9ms)  SELECT "products".* FROM "products"
# Taxon Load (1.7ms)  SELECT "taxons".* FROM "taxons" WHERE "taxons"."id" IN (9, 12, 60, 61, 20, 18, 46, 47, 15, 78, 8, 19, 14)
```

也因为是分开的两个查询, 所以, 不能在 `WHERE` 或 `ORDER BY` 的分句中使用关联表. 比如:

```ruby
[2] pry(main)> Product.preload(:taxon).where(taxons: {id: 9}).load
# ActiveRecord::StatementInvalid: PG::UndefinedTable: ERROR:  missing FROM-clause entry for table "taxons"
```

# includes

之所以大部分时候用 **includes** 就可以了, 是因为 **includes** 也能跟 **preload** 生成同样的查询语句.

```ruby
Product.includes(:taxon).load
# Product Load (33.9ms)  SELECT "products".* FROM "products"
# Taxon Load (1.7ms)  SELECT "taxons".* FROM "taxons" WHERE "taxons"."id" IN (9, 12, 60, 61, 20, 18, 46, 47, 15, 78, 8, 19, 14)
```

其实是一样的. 但除此之外, 如果有基于关联表的 `WHERE` 或者 `ORDER BY` 分句的话, 此时生成的语句不再是两条, Rails 会自动变为使用 `LEFT OUTER JOIN` 单条完成. 比如:

```ruby
Product.includes(:taxon).order("taxons.id").load
# SQL (109.8ms)  SELECT "products"."id" AS t0_r0, "products"."name" AS t0_r1, "products"."sku" AS t0_r2, "products"."taxon_id" AS t0_r3, "taxons"."id" AS t1_r0, "taxons"."name" AS t1_r1 FROM "products" LEFT OUTER JOIN "taxons" ON "taxons"."id" = "products"."taxon_id" ORDER BY taxons.id
```

除了 `LEFT JOIN` 以外, 还可以看到 `SELECT` 了两个表的所有字段. 这也造就了写报表 SQL 踩坑的点了.

因为用到了 `GROUP BY` 语句, 但其实又不需要 **includes** 自动给的额外 `SELECT` 的字段, 于是...

```ruby
ActiveRecord::StatementInvalid: PG::GroupingError: ERROR:  column "snapshots.id" must appear in the GROUP BY clause or be used in an aggregate function
```