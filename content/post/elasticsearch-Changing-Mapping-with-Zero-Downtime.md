---
title: "Elasticsearch 如何不用停机情况下完成 mapping 的修改"
date: 2016-08-13T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

> 最近使用 [searchkick](https://github.com/ankane/searchkick) 这个 Gem 才发现, 之前用的 [elasticsearch-rails](https://github.com/elastic/elasticsearch-rails) 要更灵(jian)活(lou)的多.

-------------

为了让数据可以被查询到, **Elasticsearch** 需要知道每个 field 存着的是什么样的数据, 以及这些数据是如何被索引的. 而我们也只能从 **Elasticsearch** 查找到已经存储了的索引.

**然而,** 每次更新一下某个 index, 某个 type 的 **mapping** 时候, 都得 **reindex**, 数据量不大, 文档较少的情况还可以接受, 一个操作瞬间搞定. 但是, 上到一定数量级时候, 或者 mapping 本身比较复杂的话, 每次要 reindex 可就不是几十秒, 三五分钟的事情了. 服务器总不能那么傻傻地瞎等着直到索引重新跑完, 而且是, 每一次, 每一个小改动都得这样.

**比如**, 仅仅想把某个 field 从 `string` 改成 `date` 类型, 那么, 所有已经存入的数据就会变得毫无意义. 无论如何, 最后我们还是得**重新索引(reindex)**.

其实, 并不是只是 Elasticsearch 是这样, 所有的数据库索引都一样的道理.

Elasticsearch (and Lucene) 把所有的 indices 存在不可变的 segments 中(每个 segment 就是一个迷你的倒排索引), 也就是说这些 segments **永远不会被更新.** 所谓更新某个文档, 其实只是新创建一个文档然后把旧的那个文档删掉而已. 越多的文档被更新或者创建, 也就会有越多的新的 segments 被创建. 后台则在跑着一个合并进程把几个小的 segments 合成一个大的直到所有旧的 segments 被删除.

在 Elasticsearch 中, 一个 index 可以包含有多个不同的 types. 每一个 `_type` 都有着自己的 **mapping**. 而每个 segment 中则可能包含有属于不同的 types 的文档, 这样的话, 即使仅仅是更改某个 type 的某个 field, 那么, **这个 index 的所有文档都得重新索引**.

= . = 所以, **提倡一个 index 一个 type**

官方给出了几种方式可以让在不宕机情况下完成 Mapping 的修改.

###### 添加新的 fields 可以不需要重新索引

因为 segment 只是包含着该 segment 已存有的文档数据, 这也就意味着, 只要用 [**put_mapping API**](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html) 无需 reindex 即可添加新的 fields, 包括 **multi-field**. (不过只生效于新的文档, 旧有文档对应的 fields 还是为空值)

###### reindex 的背后

数据重新索引的过程其实很简单. **首先**, 新建一个新的 index(新的 mapping 和 settings):

```javascript
curl -XPUT localhost:9200/new_index -d '
{
    "mappings": {
        "my_type": { ... new mapping definition ...}
    }
}
'
```
使用 [**scrolled search**](http://www.elasticsearch.org/guide/reference/api/search/scroll/)([**search_type=scan**](http://www.elasticsearch.org/guide/reference/api/search/search-type.html)) 从旧的 index 中获取所有的文档, 然后再使用 [**bulk API**](http://www.elasticsearch.org/guide/reference/api/bulk/)添加索引到新的 index, 完成以后删除旧的 index.

很多集成插件通常都会提供一个 `reindex()` 的方法来完成所有的这些事情.

那么问题来了, 新的 index 和旧的 index 名字是不一样的啊, 那程序也要跟着改变才行了.

###### 真正的 0 downtime

使用**索引别名(Index aliases)**让我们可以更灵活地在后台完成 reindex 的工作, 所有的一切我们的程序都可以不用操心.

[Alias](http://www.elasticsearch.org/guide/reference/api/admin-indices-aliases/) 就像 *nix 的软链接一样可以指向任意真实存在的 indices.

所以, 其实真正的工作流应该是酱紫的. **首先**, 创建一个新的 index, 命名结尾加上版本号或者时间戳:

```ruby
curl -XPUT localhost:9200/my_index_v1 -d '
{ ... mappings ... }
'
```

创建一个别名指向这个 index:

```ruby
curl -XPOST localhost:9200/_aliases -d '
{
    "actions": [
        { "add": {
            "alias": "my_index",
            "index": "my_index_v1"
        }}
    ]
}
'
```

此时, `my_index` 就可以作为 `my_index_v1` 的一个别称来与程序进行交互.

然后, 如果需要 reindex, 则可以再新建一个 index, 同样的, 在索引名的最后加上一个新的版本号, 或者时间戳:

```python
curl -XPUT localhost:9200/my_index_v2 -d '
{ ... mappings ... }
'
```

接着, 把 `my_index_v1` 的文档 reindex 到 `my_index_v2`, 完成以后把 `my_index` 这个别名指向新的 index(比如这里的`my_index_v2`):

```java
curl -XPOST localhost:9200/_aliases -d '
{
    "actions": [
        { "remove": {
            "alias": "my_index",
            "index": "my_index_v1"
        }},
        { "add": {
            "alias": "my_index",
            "index": "my_index_v2"
        }}
    ]
}
'
```

最后, 删除旧的 index,

```perl
curl -XDELETE localhost:9200/my_index_v1
```

------------

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1f6q7eob4o3j21kw0jadl8.jpg)

从 head 监控中可以看到, 旧的正常服役同时, 新的索引也正在进行中

------------

相关阅读:

[Changing Mapping with Zero Downtime](https://www.elastic.co/blog/changing-mapping-with-zero-downtime)