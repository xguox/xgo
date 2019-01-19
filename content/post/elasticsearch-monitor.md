---
title: "Elasticsearch Monitor"
date: 2016-07-08T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

##### #懒人系列
Elasticsearch 的所有监控管理查询等等操作几乎都可以在命令行里边用 `curl` 之类的工具完成, 只要记住一大堆形态各异的 url 就可以了. 如果经常用倒还好, 要是不常用, 过个一会就忘记了, 尤其对我这种上了年纪记性不大好的.

比如查集群的健康状态, 有多少 indices, 某个 index 的 mappings, 还有当前的一些配置文件. 一大车.

主要还是为了方便管控, 于是, 也就有了各种监控工具的出现. 如:

[ElasticHQ](http://www.elastichq.org/), [elasticsearch-head](https://github.com/mobz/elasticsearch-head), [elasticsearch-paramedic](https://github.com/karmi/elasticsearch-paramedic), [elasticsearch-kopf
](https://github.com/lmenezes/elasticsearch-kopf), [Bigdesk](https://github.com/lukas-vlcek/bigdesk)

这几个是比较出名的吧, 目测还有一大波相对没那么多人使用的不为人知的工具存在.

**Bigdesk** Github 上看貌似很久没更新了, 所以, 直接就跳过没安装.

用于对比, 直接就装了三个, 反正也不怎么碍着硬盘. ElasticHQ, elasticsearch-head 还有 elasticsearch-kopf. 安装方式都一样(Mac).

Elasticsearch 是用 Homebrew 装的,

```ruby
brew info elasticsearch
```

看下 Elasticsearch 的 plugin 相关信息,

```ruby
cd /usr/local/Cellar/elasticsearch/2.3.1/libexec/bin

./plugin install lmenezes/elasticsearch-kopf/2.0/v2.1.1
./plugin install mobz/elasticsearch-head
./plugin install royrusso/elasticsearch-HQ
```

使用方式都是开箱即用的直接在浏览器打开链接地址:

http://localhost:9200/_plugin/hq

http://localhost:9200/_plugin/head

http://localhost:9200/_plugin/kopf

没有深入的使用, 看上去基本功能貌似都差不多.

截图如下,

Head

![](http://ww3.sinaimg.cn/large/62fdd4d5jw1f5qa9ty5daj22801e07iu.jpg)


Kopf

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f5qa79gesbj21kw0zkafl.jpg)


ElasticHQ

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f5qa788raqj21kw0zktip.jpg)


**UPDATE:**

ElasticHQ 的 REST Editor 居然不支持自定义的 url? 不是吧, 那要来何用? = . =

elasticsearch-head 和 elasticsearch-kopf 都不错,

![](http://ww2.sinaimg.cn/large/62fdd4d5gw1f5qytg18r2j22801e0ap0.jpg)

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1f5qythq0pij22801e04fw.jpg)

综合下来感觉,
head 的功能更强大些, 自带中文 i18n, 不过没有 kopf 那样给个暗色的主题 (つд⊂)

**UPDATE:**

Elasticsearch 官方出品的 [Marvel](https://www.elastic.co/guide/en/marvel/current/installing-marvel.html), 安装稍微麻烦点, 不过都有阐明, 性能监控比较好, 作为管理工具的话还是上面几个好点吧. Basic License 免费, 更多的功能貌似是需要付费的.

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1f8bha83sk9j22801e0ano.jpg)






