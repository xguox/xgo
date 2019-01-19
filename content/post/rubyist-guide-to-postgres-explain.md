---
title: "给 Rubyist 的 Postgresql Explain 教程"
date: 2017-04-27T16:01:23+08:00
draft: false
tags: ["Ruby", "Postgres", "Translation"]
categories: ["Ruby", "Postgres", "Translation"]
---

如果想知道你的数据库查询为啥变得越来越慢了， 那没有啥比 **postgres** 的 ```EXPLAIN``` 更好使的了。

其实也没啥神秘的。 就是让 postgres 告诉咱们，它是怎么去执行这个查询的。你甚至可以进行实际查询然后比较实际的和预期性能差别。

### Sound familiar?

也许你已经见识过 **EXPLAIN**. 早在 Rails 3.2 开始， Rails 会自动 ***EXPLAIN* 那些耗时大于 500ms 的查询。

但问题是，其输出的结果有些隐秘。 举个例子，这是从 rails development blog 拿来的:

```ruby
% User.where(:id => 1).joins(:posts).explain

EXPLAIN for: SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id" WHERE "users"."id" = 1
                                  QUERY PLAN
------------------------------------------------------------------------------
 Nested Loop Left Join  (cost=0.00..37.24 rows=8 width=0)
   Join Filter: (posts.user_id = users.id)
   ->  Index Scan using users_pkey on users  (cost=0.00..8.27 rows=1 width=4)
         Index Cond: (id = 1)
   ->  Seq Scan on posts  (cost=0.00..28.88 rows=8 width=4)
         Filter: (posts.user_id = 1)
(6 rows)
```

所以， 这是什么鬼？

在这篇文章我们将会分析类似这样的输出结果，及其给我们的 Ruby Web 开发所带来的影响。

### The syntax

如果正在使用的是 Rails, 那么可以在任意 ActiveRecord 查询后面加上 ```.explain```。

```ruby
> User.where(id: 1).explain
  User Load (10.0ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1  [["id", 1]]
=> EXPLAIN for: SELECT "users".* FROM "users" WHERE "users"."id" = $1 [["id", 1]]
                                QUERY PLAN
--------------------------------------------------------------------------
 Index Scan using users_pkey on users  (cost=0.29..8.30 rows=1 width=812)
   Index Cond: (id = 1)
(2 rows)
```

虽然，```EXPLAIN``` 方法使用起来很方便， 但是， 它并没有给你一些我们可以在 postgres 中可以用到的高级选项。

要在 postgres 中直接使用 **EXPLAIN**， 首先用 ```psql -d yourdb``` 进入到你的数据库中，然后再执行下面这条语句:

```ruby
EXPLAIN SELECT * FROM users WHERE id=1;
                                QUERY PLAN
--------------------------------------------------------------------------
 Index Scan using users_pkey on users  (cost=0.29..8.30 rows=1 width=812)
   Index Cond: (id = 1)
(2 rows)
```

这样我们就能得到一些关于 postgres 如何执行查询的信息，包括最少需要做多少工作来完成这个查询。

只用 **EXPLAIN** 的话并不会真正的执行这条语句，想要真正执行语句， 并给出预期与实际运行结果的对比则需要用 ```EXPLAIN ANALYZE```

```ruby
EXPLAIN ANALYZE SELECT * FROM users WHERE id=1;
                               QUERY PLAN
------------------------------------------------------------------------------------
 Index Scan using users_pkey on users  (cost=0.29..8.30 rows=1 width=812) (actual time=0.043..0.044 rows=1 loops=1)
   Index Cond: (id = 1)
 Total runtime: 0.117 ms
```

### Interpreting the output

Postgres 很聪明的呢，他可以找到最高效的方式来执行你的查询。换句话说，他会生成一个「查询计划」，然后 ```explain```只是负责把这个计划输出出来。

考虑以下情况:

```ruby
# explain select * from users order by created_at limit 10;
                               QUERY PLAN
-------------------------------------------------------------------------
 Limit  (cost=892.98..893.01 rows=10 width=812)
   ->  Sort  (cost=892.98..919.16 rows=104 width=812)
         Sort Key: created_at
         ->  Seq Scan on users  (cost=0.00..666.71 rows=104 width=812)
(4 rows)
```

这个输出由两部分组成:

1. 「节点列表」(node list) 描述了执行该查询时候所发生的一系列动作
2. 性能预估，描述了这个列表中的每个动作的耗时。

### The node list

去掉所有的性能评估以后， 也就只剩下节点列表了，如下所示:

```ruby
Limit
   ->  Sort (Sort Key: created_at)
         ->  Seq Scan on users
```

这有点像是一个 Postgres 写的「程序」用来执行查询操作。 其中包含了三个动作， "limit", "sort" 和 "seq scan"。 子节点中的输出结果会被向上传送到父节点。 用 Ruby 的话来说就是:

```ruby
all_users.sort(:created_at).limit(10)
```

Postgres 会在查询计划中使用许多不同的动作。 但你并不需要掌握所有的这些动作的含义，记住下面这几个就好了:

- Index Scan: 通过索引来获取记录。 类似在 Ruby 的 Hash 中查找某一项。
- Seq Scan: 通过遍历结果集来获取记录。
- Filter: 从结果集中选择那些匹配某条件的记录。
- Sort: 对结果集排序
- Aggregate: 当执行像 count, max, min 这类操作时候会用到
- Bitmap Heap Scan: Uses a bitmap to represent matching records. Operations like and-ing and or-ing can sometimes be done more easily to the bitmap than to the actual records.

[https://github.com/digoal/blog/blob/master/201702/20170221_02.md](https://github.com/digoal/blog/blob/master/201702/20170221_02.md)

### Performance estimates

节点列表中， 每个节点都有一组性能评估。一般长这样子的:

```ruby
Limit  (cost=892.98..893.01 rows=10 width=812)
```

这些数字代表意思:

- **Cost**: 该动作的执行成本。它没有具体的单位， 只能和其他同样是 **Cost** 的数字进行对比
- **Rows**: 执行该操作一共需要遍历多少行记录。
- **Width**: 每一行大概有多少字节。

最常用到的参数应该是 **Rows**。它可以清楚的让我们知道查询的性能。 如果这个值等于 1， 那么该查询性能杠杠的。 如果发现这个值等于表中记录的总数，那么这个查询性能就会随着表的增大变的越来越慢了。

### Actual performance values

使用 **EXPLAIN ANALYZE** 时， 会真正的去执行查询，然后就会出现两组数字。前一组是像上面的预估值，后一组则是实际值。

```ruby
# explain analyze select * from users order by created_at limit 10;
                                                       QUERY PLAN
------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=892.98..893.01 rows=10 width=812) (actual time=22.443..22.446 rows=10 loops=1)
   ->  Sort  (cost=892.98..919.16 rows=10471 width=812) (actual time=22.441..22.443 rows=10 loops=1)
         Sort Key: created_at
         Sort Method: top-N heapsort  Memory: 31kB
         ->  Seq Scan on users  (cost=0.00..666.71 rows=10471 width=812) (actual time=0.203..15.221 rows=10472 loops=1)
 Total runtime: 22.519 ms
(6 rows)
```

- Actual time: 执行该动作耗费的时长(ms)
- Rows: 执行该操作实际遍历了多少行记录。
- Loops: 有时候某个动作会发生不止一次， 此时， loops 会大于 1

### More verbose output

默认情况下 **EXPLAIN** 给的是一个比较精简的版本。其实， 你可以让从他那得到更多更详细的信息。甚至可以让他的输出信息格式化为 JSON 或者 YAML.

```yaml
# EXPLAIN (ANALYZE, FORMAT YAML) select * from users order by created_at limit 10;
                QUERY PLAN
------------------------------------------
 - Plan:                                 +
     Node Type: "Limit"                  +
     Startup Cost: 892.98                +
     Total Cost: 893.01                  +
     Plan Rows: 10                       +
     Plan Width: 812                     +
     Actual Startup Time: 12.945         +
     Actual Total Time: 12.947           +
     Actual Rows: 10                     +
     Actual Loops: 1                     +
     Plans:                              +
       - Node Type: "Sort"               +
         Parent Relationship: "Outer"    +
         Startup Cost: 892.98            +
         Total Cost: 919.16              +
         Plan Rows: 10471                +
         Plan Width: 812                 +
         Actual Startup Time: 12.944     +
         Actual Total Time: 12.946       +
         Actual Rows: 10                 +
         Actual Loops: 1                 +
         Sort Key:                       +
           - "created_at"                +
         Sort Method: "top-N heapsort"   +
         Sort Space Used: 31             +
         Sort Space Type: "Memory"       +
         Plans:                          +
           - Node Type: "Seq Scan"       +
             Parent Relationship: "Outer"+
             Relation Name: "users"      +
             Alias: "users"              +
             Startup Cost: 0.00          +
             Total Cost: 666.71          +
             Plan Rows: 10471            +
             Plan Width: 812             +
             Actual Startup Time: 0.008  +
             Actual Total Time: 5.823    +
             Actual Rows: 10472          +
             Actual Loops: 1             +
   Triggers:                             +
   Total Runtime: 13.001
(1 row)
```

还可以这样，```EXPLAIN (ANALYZE, VERBOSE, FORMAT YAML) SELECT ...```

### Visualization tools

Explain 会生成一大堆的输出。 尤其当查询比较复杂的时候，更是难以解析。

有一些免费的工具可以帮上忙。他们会帮助解析并生成一些漂亮的图表给你。甚至会给出一些标记指出潜在的性能问题。[比如这个](http://tatiyants.com/pev/#/plans/new)

![](http://www.rubyletter.com/images/2016/12/explain.png)

### Examples

把上面提到的汇总一起举个例子。你是否发现某一些 Rails 的通用做法会生成一些性能不佳的数据库查询?

#### The counting conundrum

下面的用法应该是非常常见的吧:

```ruby
Total Faults <%= Fault.count %>
```

对应生成的 SQL 语句如下:

```ruby
select count(*) from faults;
```

给 **EXPLAIN** 一下，看看发生了些什么事情。

```ruby
# explain select count(*) from faults;
                            QUERY PLAN
-------------------------------------------------------------------
 Aggregate  (cost=1840.31..1840.32 rows=1 width=0)
   ->  Seq Scan on faults  (cost=0.00..1784.65 rows=22265 width=0)
(2 rows)

```

Woah! 简单的一个 count 查询，竟然遍历了  22,265 行， 也就是整个表。 在 Postgres , count 都是全表查的。

#### The sorting scandal

下面这个也常见吧， 根据某个字段进行排序。 比如:

```ruby
Fault.order("created_at desc").limit(10)
```

其实你只想要 10 条记录而已。 但， 为了得到这 10 条记录， 你需要对整个表进行排序。 从下面可以看出 **Sort** 操作遍历了 22,265 行记录.

```ruby
# explain select * from faults order by created_at limit 10;
                                 QUERY PLAN
----------------------------------------------------------------------------
 Limit  (cost=2265.79..2265.81 rows=10 width=1855)
   ->  Sort  (cost=2265.79..2321.45 rows=22265 width=1855)
         Sort Key: created_at
         ->  Seq Scan on faults  (cost=0.00..1784.65 rows=22265 width=1855)
```

通过添加索引，我们就可以把 ***Sort* 这个操作换成更高效的 **Index Scan**

```ruby
# CREATE INDEX index_faults_on_created_at ON faults USING btree (created_at);
CREATE INDEX

# explain select * from faults order by created_at limit 10;
                                               QUERY PLAN
---------------------------------------------------------------------------------------------------------
 Limit  (cost=0.29..2.66 rows=10 width=1855)
   ->  Index Scan using index_faults_on_created_at on faults  (cost=0.29..5288.04 rows=22265 width=1855)
(2 rows)
```

### Conclusion

强烈建议多一些使用 ```EXPLAIN``` 这个命令， 这样， 当数据越来越大，时候不至于措手无策。

译自: [A Rubyist's Guide to Postgresql's Explain](http://www.rubyletter.com/blog/2017/03/13/rubyist-guide-to-postgres-explain.html)