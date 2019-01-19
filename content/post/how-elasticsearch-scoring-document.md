---
title: "Elasticsearch 的分数(_score)是怎么计算得出"
date: 2016-11-04T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

上次写了关于 [Elasticsearch 如何分词索引](http://xguox.me/elasticsearch-custom-analyzer.html), 接着继续写 Elasticsearch 怎么计算搜索结果的得分(_score).

**Elasticsearch 默认是按照文档与查询的相关度(匹配度)的得分倒序返回结果的.  得分 (_score) 就越大,  表示相关性越高.**

所以, 相关度是啥? 分数又是怎么计算出来的? (全文检索和结构化的 SQL 查询不太一样, 虽然看起来结果比较'飘忽', 但也是可以追根问底的)

------------

在 Elasticsearch 中, 标准的算法是  **Term Frequency/Inverse Document Frequency, 简写为 TF/IDF**, (刚刚发布的 5.0 版本, 改为了据说更先进的 BM25 算法)

[**->** Elasticsearch 5.X(Lucene 6) 的 BM25 相关度算法](http://xguox.me/how-elasticsearch-5-scoring-with-bm25.html)

##### Term Frequency

某单个关键词(term) 在某文档的某字段中出现的频率次数, 显然, 出现频率越高意味着该文档与搜索的相关度也越高

具体计算公式是 **tf(q in d) = sqrt(termFreq)**

另外, 索引的时候可以做一些设置, "index_options": "docs" 的情况下, 只考虑 term 是否出现(命中), 不考虑出现的次数.

```json
PUT /my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "text": {
          "type":          "string",
          "index_options": "docs"
        }
      }
    }
  }
}
```

##### Inverse document frequency

某个关键词(term) 在索引(单个分片)之中出现的频次. 出现频次越高, 这个词的相关度越低. 相对的, 当某个关键词(term)在一大票的文档下面都有出现, 那么这个词在计算得分时候所占的比重就要比那些只在少部分文档出现的词所占的得分比重要低. 说的那么长一句话, 用人话来描述就是 **"物以稀为贵"**, 比如, '的', '得', 'the' 这些一般在一些文档中出现的频次都是非常高的, 因此, 这些词占的得分比重远比特殊一些的词(如'Solr', 'Docker', '哈苏')占比要低,

具体计算公式是 **idf = 1 + ln(maxDocs/(docFreq + 1))**

##### Field-length Norm

字段长度, 这个字段长度越短, 那么字段里的每个词的相关度也就越大. 某个关键词(term) 在一个短的句子出现, 其得分比重比在一个长句子中出现要来的高.

具体计算公式是 **norm = 1/sqrt(numFieldTerms)**

默认每个 analyzed 的 string 都有一个 norm 值, 用来存储该字段的长度,

用 "norms": { "enabled": false } 关闭以后, 评分时, 不管文档的该字段长短如何, 得分都一样.

```json
PUT /my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "text": {
          "type": "string",
          "norms": { "enabled": false }
        }
      }
    }
  }
}
```

##### 最后的得分是三者的乘积 tf * idf * norm

以上描述的是最原始的针对**单个关键字(term)**的搜索. 如果是有**多个搜索关键词(terms)**的时候, 还要用到的 **Vector Space Model**

如果查询复杂些, 或者用到一些修改了分数的查询, 或者索引时候修改了字段的权重, 比如 **function_score** 之类的,计算方式也就又更复杂一些.

##### [Explain](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/search-explain.html)

**看上去 TF/IDF 的算法已经一脸懵逼吓跑人了, 不过其实, 用 Explain 跑一跑也没啥, 虽然各种开方, 自然对数的, Google一个科学计算器就是了.**

**举个例子**

```javascript
/*先删掉索引, 如果有的话*/
curl -XDELETE 'http://localhost:9200/blog'

curl -XPUT 'http://localhost:9200/blog/' -d '
{
  "mappings": {
     "post": {
        "properties": {
           "title": {
              "type": "string",
              "analyzer": "standard",
              "term_vector": "yes"
           }
        }
     }
  }
}'
```

存入一些文档 (Water 随手加进去测试的.)

```javascript
curl -s -XPOST localhost:9200/_bulk -d '
{ "create": { "_index": "blog", "_type": "post", "_id": "1" }}
{ "title": "What is the best water temperature, Mr Water" }
{ "create": { "_index": "blog", "_type": "post", "_id": "2" }}
{ "title": "Water no symptoms" }
{ "create": { "_index": "blog", "_type": "post", "_id": "3" }}
{ "title": "Did Vitamin B6 alone work for you? Water?" }
{ "create": { "_index": "blog", "_type": "post", "_id": "4" }}
{ "title": "The ball drifted on the water." }
{ "create": { "_index": "blog", "_type": "post", "_id": "5" }}
{ "title": "No water no food no air" }
'
```

bulk insert 以后先用 Kopf 插件输出看一下, 5 个文档并不是平均分配在 5 个分片的, 其中, 编号为 2 的这个分片里边有两个文档, 其中编号为 0 的那个分片是没有分配文档在里面的.

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1f9fb44fanfj227w17un6g.jpg)

接下来, 搜索的同时 **explain**

原本输出的 json 即使加了 pretty 也很难看, 换成 yaml 会好不少

```json
curl -XGET "http://127.0.0.1:9200/blog/post/_search?explain&format=yaml" -d '
{
  "query": {
    "term": {
      "title": "water"
    }
  }
}'
```

输出如图(json)

![](http://ww3.sinaimg.cn/large/62fdd4d5gw1f9fb43tttmj21hu1dyah4.jpg)

可以看到五个文档都命中了这个查询, 注意看每个文档的 **_shard**

整个输出 yml 太长了, 丢到最后面, 只截取了其中一部分, 如图,

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f9fb44r1ukj217018gtft.jpg)

返回排名第一的分数是 **_score: 0.2972674**, _shard(2),

**"weight(title:water in 0) [PerFieldSimilarity], result of:"** 这里的 0 不是 _id, 只是 Lucene 的一个内部文档 ID, 可以忽略.

排名第一和第二的两个文档刚好是在同一个分片的, 所以跟另外三个的返回结果有些许不一样, 主要就是多了一个 **queryWeight**, 里面的 **queryNorm** 只要在同一分片下, 都是一样的, 总而言之, 这个可以忽略(至少目前这个例子可以忽略)

只关注 **fieldWeight**, 排名第一和第二的的 tf 都是 1,

在 **idf(docFreq=2, maxDocs=2)** 中,  docFreq 和 maxDocs 都是针对单个分片而言, 2号分片一共有 2个文档(maxDocs), 然后命中的文档也是两个(docFreq).

所以 idf 的得分, 根据公式, **1 + ln(maxDocs/(docFreq + 1))** 是 0.59453489189

最后 **fieldNorm**, 这个 field 有三个词, 所以是 1/sqrt(3), 但是按官方给的这个公式怎么算都不对, 不管哪个文档. 后来查了一下, 说是 Lucene 存这个 lengthNorm 数据时候都是用的 1 byte来存, 所以不管怎么着都会丢掉一些精度. 呵呵哒了 = . =

最后的最后, 总得分 = 1 * 0.5945349 * 0.5 = 0.2972674.

同理其他的几个文档也可以算出这个得分, 只是都要被 fieldNorm 的精度问题蛋疼一把.

```yml
took: 3
timed_out: false
_shards:
  total: 5
  successful: 5
  failed: 0
hits:
  total: 5
  max_score: 0.2972674
  hits:
  - _shard: 2
    _node: "aVHxi_d2TYq40ISddYZF-A"
    _index: "blog"
    _type: "post"
    _id: "2"
    _score: 0.2972674
    _source:
      title: "Water no symptoms"
    _explanation:
      value: 0.2972674
      description: "sum of:"
      details:
      - value: 0.2972674
        description: "weight(title:water in 0) [PerFieldSimilarity], result of:"
        details:
        - value: 0.2972674
          description: "score(doc=0,freq=1.0), product of:"
          details:
          - value: 0.99999994
            description: "queryWeight, product of:"
            details:
            - value: 0.5945349
              description: "idf(docFreq=2, maxDocs=2)"
              details: []
            - value: 1.681987
              description: "queryNorm"
              details: []
          - value: 0.29726744
            description: "fieldWeight in 0, product of:"
            details:
            - value: 1.0
              description: "tf(freq=1.0), with freq of:"
              details:
              - value: 1.0
                description: "termFreq=1.0"
                details: []
            - value: 0.5945349
              description: "idf(docFreq=2, maxDocs=2)"
              details: []
            - value: 0.5
              description: "fieldNorm(doc=0)"
              details: []
      - value: 0.0
        description: "match on required clause, product of:"
        details:
        - value: 0.0
          description: "# clause"
          details: []
        - value: 1.681987
          description: "_type:post, product of:"
          details:
          - value: 1.0
            description: "boost"
            details: []
          - value: 1.681987
            description: "queryNorm"
            details: []
  - _shard: 2
    _node: "aVHxi_d2TYq40ISddYZF-A"
    _index: "blog"
    _type: "post"
    _id: "4"
    _score: 0.22295055
    _source:
      title: "The ball drifted on the water."
    _explanation:
      value: 0.22295056
      description: "sum of:"
      details:
      - value: 0.22295056
        description: "weight(title:water in 1) [PerFieldSimilarity], result of:"
        details:
        - value: 0.22295056
          description: "score(doc=1,freq=1.0), product of:"
          details:
          - value: 0.99999994
            description: "queryWeight, product of:"
            details:
            - value: 0.5945349
              description: "idf(docFreq=2, maxDocs=2)"
              details: []
            - value: 1.681987
              description: "queryNorm"
              details: []
          - value: 0.22295058
            description: "fieldWeight in 1, product of:"
            details:
            - value: 1.0
              description: "tf(freq=1.0), with freq of:"
              details:
              - value: 1.0
                description: "termFreq=1.0"
                details: []
            - value: 0.5945349
              description: "idf(docFreq=2, maxDocs=2)"
              details: []
            - value: 0.375
              description: "fieldNorm(doc=1)"
              details: []
      - value: 0.0
        description: "match on required clause, product of:"
        details:
        - value: 0.0
          description: "# clause"
          details: []
        - value: 1.681987
          description: "_type:post, product of:"
          details:
          - value: 1.0
            description: "boost"
            details: []
          - value: 1.681987
            description: "queryNorm"
            details: []
  - _shard: 3
    _node: "aVHxi_d2TYq40ISddYZF-A"
    _index: "blog"
    _type: "post"
    _id: "1"
    _score: 0.13561106
    _source:
      title: "What is the best water temperature, Mr Water"
    _explanation:
      value: 0.13561106
      description: "sum of:"
      details:
      - value: 0.13561106
        description: "weight(title:water in 0) [PerFieldSimilarity], result of:"
        details:
        - value: 0.13561106
          description: "fieldWeight in 0, product of:"
          details:
          - value: 1.4142135
            description: "tf(freq=2.0), with freq of:"
            details:
            - value: 2.0
              description: "termFreq=2.0"
              details: []
          - value: 0.30685282
            description: "idf(docFreq=1, maxDocs=1)"
            details: []
          - value: 0.3125
            description: "fieldNorm(doc=0)"
            details: []
      - value: 0.0
        description: "match on required clause, product of:"
        details:
        - value: 0.0
          description: "# clause"
          details: []
        - value: 3.2588913
          description: "_type:post, product of:"
          details:
          - value: 1.0
            description: "boost"
            details: []
          - value: 3.2588913
            description: "queryNorm"
            details: []
  - _shard: 1
    _node: "aVHxi_d2TYq40ISddYZF-A"
    _index: "blog"
    _type: "post"
    _id: "5"
    _score: 0.11506981
    _source:
      title: "No water no food no air"
    _explanation:
      value: 0.11506981
      description: "sum of:"
      details:
      - value: 0.11506981
        description: "weight(title:water in 0) [PerFieldSimilarity], result of:"
        details:
        - value: 0.11506981
          description: "fieldWeight in 0, product of:"
          details:
          - value: 1.0
            description: "tf(freq=1.0), with freq of:"
            details:
            - value: 1.0
              description: "termFreq=1.0"
              details: []
          - value: 0.30685282
            description: "idf(docFreq=1, maxDocs=1)"
            details: []
          - value: 0.375
            description: "fieldNorm(doc=0)"
            details: []
      - value: 0.0
        description: "match on required clause, product of:"
        details:
        - value: 0.0
          description: "# clause"
          details: []
        - value: 3.2588913
          description: "_type:post, product of:"
          details:
          - value: 1.0
            description: "boost"
            details: []
          - value: 3.2588913
            description: "queryNorm"
            details: []
  - _shard: 4
    _node: "aVHxi_d2TYq40ISddYZF-A"
    _index: "blog"
    _type: "post"
    _id: "3"
    _score: 0.095891505
    _source:
      title: "Did Vitamin B6 alone work for you? Water?"
    _explanation:
      value: 0.095891505
      description: "sum of:"
      details:
      - value: 0.095891505
        description: "weight(title:water in 0) [PerFieldSimilarity], result of:"
        details:
        - value: 0.095891505
          description: "fieldWeight in 0, product of:"
          details:
          - value: 1.0
            description: "tf(freq=1.0), with freq of:"
            details:
            - value: 1.0
              description: "termFreq=1.0"
              details: []
          - value: 0.30685282
            description: "idf(docFreq=1, maxDocs=1)"
            details: []
          - value: 0.3125
            description: "fieldNorm(doc=0)"
            details: []
      - value: 0.0
        description: "match on required clause, product of:"
        details:
        - value: 0.0
          description: "# clause"
          details: []
        - value: 3.2588913
          description: "_type:post, product of:"
          details:
          - value: 1.0
            description: "boost"
            details: []
          - value: 3.2588913
            description: "queryNorm"
            details: []
```