---
title: "Elasticsearch 5/6 的 Web UI 管理工具"
date: 2018-08-01T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

许久没仔细使用 Elasticsearch 以后, 发现版本号已经从之前用的 2.X 升级到 6.3, 发布 5 的时候有关注过一会. 之前那堆管理工具打开一看也都好久没更新了, 可能是因为 [**Site plugins are not supported in Elasticsearch 5.0**](https://www.elastic.co/blog/running-site-plugins-with-elasticsearch-5-0) 了吧.

新发现一个叫 [dejavu](https://github.com/appbaseio/dejavu) 更新的还挺活跃, 自己 clone 下来 yarn 安装一下就可以跑起来了, 因为之前写了一些前端也就「标配」了 yarn 在系统里边. 又或者直接用 chrome 的拓展([elasticsearch-head](https://github.com/mobz/elasticsearch-head) 也可以)

在 **elasticsearch.yml** 还得修改一下跨域限制

```
http.cors.allow-origin: "http://localhost:1358"
http.cors.enabled: true
http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization
http.cors.allow-credentials: true
```

不爽的地方就是一次只能连着一个 index, 然而我又[属于 index 党而不是 type 党](https://www.elastic.co/blog/index-vs-type) = . = 看在 UI 比 elasticsearch-head 好看一些先用着
