---
title: "Elasticsearch 5.X(Lucene 6) 的 BM25 相关度算法"
date: 2016-12-19T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

上次写了关于 [Elasticsearch 的分数(_score)是怎么计算得出](http://xguox.me/how-elasticsearch-scoring-document.html), 不过那是在 **Elasticsearch 5.0** 以前的版本用的, **Elasticsearch** [前不久发布了 5.0 版本](https://www.elastic.co/blog/elasticsearch-5-0-0-released), 基于 Lucene 6, 默认使用了 [BM25](https://en.wikipedia.org/wiki/Okapi_BM25) 评分算法.

**BM25** 的 **BM 是缩写自 Best Match**, 25 貌似是经过 25 次迭代调整之后得出的算法. 它也是基于 **TF/IDF** 进化来的. [Wikipedia 那个公式](https://en.wikipedia.org/wiki/Okapi_BM25)看起来很吓唬人, 尤其是那个求和符号, 不过分解开来也是比较好理解的.

总体而言, 主要还是分三部分, **TF - IDF - Document Length**

![](https://ww2.sinaimg.cn/large/62fdd4d5jw1fauy0sa5z2j20go023jrd.jpg)

IDF 还是和[之前的一样](http://xguox.me/how-elasticsearch-scoring-document.html#inverse-document-frequency). 公式 IDF(q) = 1 + ln(maxDocs/(docFreq + 1))

**f(q, D) 是 tf(term frequency)**

**\|d\| 是文档的长度, avgdl 是平均文档长度.**

先不看 IDF 和 Document Length 的部分, 变成 **tf * (k + 1) / (tf + k)**,

相比传统的 TF/IDF (**tf(q in d) = sqrt(termFreq)**) 而言,
 **BM25 抑制了 tf 对整体评分的影响程度**, 虽然同样都是增函数, 但是, BM25 中, tf 越大, 带来的影响无限趋近于 (k + 1), 这里 k 值通常取 [1.2, 2], 而传统的 **TF/IDF** 则会没有临界点的无限增长.

而**文档长度的影响**, 同样的, 可以看到, 命中搜索词的情况下, 文档越短, 相关性越高, 具体影响程度又可以由公式中的 b 来调整, 当设值为 0 的时候, 就跟之前 '[TF/IDF](http://xguox.me/how-elasticsearch-scoring-document.html#field-length-norm)' 那篇提到的 `"norms": { "enabled": false }` 一样, 忽略文档长度的影响.

综合起来,

k = 1.2

b = 0.75

**idf * (tf * (k + 1)) / (tf + k * (1 - b + b * (\|d\|/avgdl)))**

最后再对所有 term 求和. 就是 Elasticsearch 5 中一般查询的得分了.

### Related:

[http://opensourceconnections.com/blog/2015/10/16/bm25-the-next-generation-of-lucene-relevation/](http://opensourceconnections.com/blog/2015/10/16/bm25-the-next-generation-of-lucene-relevation/)
