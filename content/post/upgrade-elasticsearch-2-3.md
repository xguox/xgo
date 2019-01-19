---
title: "Upgrade Elasticsearch to 2.3"
date: 2016-04-12T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

正式环境和本地一直跑的 1.X, 最近在用 Elasticsearch 的 Aggregations 功能, 发现有一些个语法在 2.X 版本变了, 想来也要跟上潮流了. 于是着手各种升级事宜.

官方有专门的 [Rolling upgrades 链接](https://www.elastic.co/guide/en/elasticsearch/reference/current/rolling-upgrades.html#upgrade-node)讲如何升级的, 不过, 对于目前在用的情况来说, 貌似也用不上那么麻烦. 好吧, 其实有点谈不上升级, 感觉像是直接卸载了重装差不多 = . =

{% highlight ruby %}
# Ubuntu
sudo service elasticsearch stop
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-2.3.1.deb
sudo dpkg -P elasticsearch
sudo dpkg -i elasticsearch-2.3.1.deb
sudo service elasticsearch start
{% endhighlight %}

理论上应该无痛可以跑起来的, 不过, 有一台服务器上, 安装好了以后, Elasticsearch 一直跑不起来
`sudo service elasticsearch start` 一直 fail~fail~fail

elasticsearch.log 又啥也没输出. 折腾了半天才发现原来是 Java 的版本没跟上. 本地的 Java 8 和另一台 Java 7 都没事, 唯独这台, `java -version`

```ruby
java version "1.6.0_65"
Java(TM) SE Runtime Environment (build 1.6.0_65-b14-468-11M4833)
Java HotSpot(TM) 64-Bit Server VM (build 20.65-b04-468, mixed mode)
```

Elasticsearch 的官方 requirements 说最低是要 Java 7, 像之前一直跑的这个 Java 6 很可能会有各种已知的 bugs, 虽然, 1.X 还是能在这种情况下跑起来, 不过升级到 2.X 就歇菜了.

------

升级完以后查看了一下状态

`curl -XGET http://localhost:9200/_cluster/health\?pretty\=true`

发现一直以来 Elasticsearch 的这个 status 都是黄的, 印象中好像没绿过

```ruby
➜ curl -XGET http://localhost:9200/_cluster/health\?pretty\=true
{
  "cluster_name" : "elasticsearch_xguox",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 6,
  "active_shards" : 6,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 1,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 85.71428571428571
}
```

elasticsearch.yml 配置文件中有一段注释: 修改 `index.number_of_replicas: 0` 然后 reindex 就绿了.

```ruby
# Note, that for development on a local machine, with small indices, it usually
# makes sense to "disable" the distributed features:
#
#index.number_of_shards: 1
#index.number_of_replicas: 0

# These settings directly affect the performance of index and search operations
# in your cluster. Assuming you have enough machines to hold shards and
# replicas, the rule of thumb is:
#
# 1. Having more *shards* enhances the _indexing_ performance and allows to
#    _distribute_ a big index across machines.
# 2. Having more *replicas* enhances the _search_ performance and improves the
#    cluster _availability_.
#
# The "number_of_shards" is a one-time setting for an index.
#
# The "number_of_replicas" can be increased or decreased anytime,
# by using the Index Update Settings API.
#
```

```ruby
➜ curl -XGET http://localhost:9200/_cluster/health\?pretty\=true
{
  "cluster_name" : "elasticsearch_xguox",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 6,
  "active_shards" : 6,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

还有就是, 查看 elasticsearch.log 的时候经常看到类似这样的信息,

```ruby
high disk watermark [90%] exceeded on [VopDVS8uRPurOowYR78xkQ][Marvel Boy][/usr/local/var/elasticsearch/elasticsearch_xguox/nodes/0] free: 23gb[9.9%], shards will be relocated away from this node

```

也是在 配置文件 `elasticsearch.yml` 添加下面几句就搞定了:

```ruby
cluster.routing.allocation.disk.threshold_enabled: true

cluster.routing.allocation.disk.watermark.low: .97

cluster.routing.allocation.disk.watermark.high: .99
```

------

BTW, 现在都流行飙版本号了?

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f2ua8tdd6mj20h80imab9.jpg)

React 直接跳到 15.X, 这...

---
<!--more-->---
<!--more-->

Update: 2016.04.23
参加完 Elasticsearch 的线下沙龙涨了点点知识回来, 唉, 发现人家那才真的叫玩大数据啊, 各种 Spark, Hadoop, 几十几百台的机器, 上 TB 级别的数据量.

所以, 难怪我这种说升级说的那么轻松的.

Elasticsearch 升级 2.X 前用官方插件检测一下是否兼容
[elasticsearch-migration](https://github.com/elastic/elasticsearch-migration)

还有其他一些建议, 顺带贴下图纪念一下手机摄像头已坏 = . =

![](http://ww2.sinaimg.cn/large/62fdd4d5gw1f36zhmyhqdj241s31cu10.jpg)

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f36zh295l1j241s31c1l1.jpg)

![](http://ww1.sinaimg.cn/large/62fdd4d5gw1f36zgw6243j241s31cx6u.jpg)

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f36zgnrk6tj241s31c1l1.jpg)

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f36zgi9lc4j241s31c4qv.jpg)

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1f36zg9t49gj241s31c4qv.jpg)

--------

##### Related:

[Elasticsearch 开箱笔记](http://xguox.me/elasticsearch-ik-mmseg-homebrew-ubuntu.html)

[Elasticsearch on Rails](http://xguox.me/elasticsearch-rails.html)

[Elasticsearch More Like This 搜索](http://xguox.me/elasticsearch-more-like-this.html)

[Elasticsearch Aggregations 聚合分析](http://xguox.me/elasticsearch-aggregations.html)

[Elasticsearch Scroll (Ruby)](http://xguox.me/elasticsearch-scroll.html)

