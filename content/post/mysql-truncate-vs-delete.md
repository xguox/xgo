---
title: "MySQL 的 TRUNCATE"
date: 2015-09-06T16:01:23+08:00
draft: false
tags: ["MySQL"]
categories: ["MySQL"]
---

```ruby
TRUNCATE [TABLE] tbl_name

```

TRUNCATE 语句可以用来清空一个表格里面的数据.

跟使用 `DELETE` 语句来删除所有行相似, 又或者也跟 `DROP TABLE` + `CREATE TABLE` 连用的效果相似.

为了达到更高的性能, 它绕过了那些删除数据的 DML(数据操纵语言).
尽管跟 `DELETE` 相似, 不过, 它是被归类为 DDL(数据定义语言), 在 MySQL 5.6 中,  和 `DELETE` 语句的区别如下:

1. Truncate 操作是直接 drop 了以后重新 create 的, 比一条一条的删除快得多, 尤其数据表非常大的时候更明显.
2. Truncate 操作是不能被回滚的.
3. 被锁着表不能执行 Truncate 操作
4. Truncate 操作不会返回一个数值表示有多少行被删除了. 通常返回的结果都是 `0 rows affected`
5. 所有的 `AUTO_INCREMENT` 的值都会被重置为起始值.
6. Truncate 操作不会调用 ON DELETE 触发器 (对触发器不熟悉 = . =)
7. 如果数据表有外键约束是不能执行 Truncate 操作的.
