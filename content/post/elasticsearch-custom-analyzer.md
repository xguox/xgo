---
title: "Elasticsearch analysis & 自定义 analyzers"
date: 2016-08-01T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

当一个 document 被索引时, 通常是对应每个 field 都生成一个**倒排索引(Inverted Index)**用于作为存储的数据结构, 关于倒排索引, 推荐炮哥之前写的[一篇文章](https://ruby-china.org/topics/23447)可以结合参考理解. 每个 field 的倒排索引是由**「对应」**于这个 field 的那些词语(term)所组成. 从而, 搜索的时候, 就可查到某个 document 是否含有(或者说命中)某些 terms, 进而返回所有命中查找词的 documents.

这里强调的**「对应」**其实就是 **Analyzers** 在支持着.

----------

Elasticsearch 的[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html):

> Analyzers are composed of a single [Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html) and zero or more [TokenFilters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html). The tokenizer may be preceded by one or more [CharFilters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html).

**Analyzers** 是由一个 **Tokenizer** 和任意个数的 **TokenFilters** 组成. 而在把需要处理的字符串传给 Tokenizer 之前需要经过一个或多个的 CharFilters(Character filters) 做预处理.

**Elasticsearch(2.3)** 默认给了我们八种 **Analyzers(standard, simple, whitespace, stop, keyword, pattern, language, snowball)** 开箱即用.  此外, 还提供给了我们 **3 种 CharFilters, 12 种 Tokenizer, 以及一大堆 TokenFilters** 用于自定义 Analyzers.

----------


说半天, 都是很抽象的东西. 下面用一些栗子说明.

新建索引, 并设置好 mappings

```javascript
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

存入数据如下,

```ruby
curl -XPUT 'http://localhost:9200/blog/post/1?pretty=true' -d '
{
    "title": "the Good Cats & the Good Dogs!"
}
'
```

使用 **[termvector](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html)** 来查看 title 这个 field 的数据是如何存储的(倒排索引)

```javascript
curl -XGET 'http://localhost:9200/blog/post/1/_termvector?fields=title&pretty=true'

{
  "_index" : "blog",
  "_type" : "post",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "took" : 1,
  "term_vectors" : {
    "title" : {
      "field_statistics" : {
        "sum_doc_freq" : 4,
        "doc_count" : 1,
        "sum_ttf" : 6
      },
      "terms" : {
        "cats" : {
          "term_freq" : 1
        },
        "dogs" : {
          "term_freq" : 1
        },
        "good" : {
          "term_freq" : 2
        },
        "the" : {
          "term_freq" : 2
        }
      }
    }
  }
}

```

因为在定义 mappings 的时候, title 用的 analyzer 是 standard, [官方给的 standard](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-tokenizer.html) 是由以下几个部分组成的

> An analyzer of type standard is built using the Standard Tokenizer with the Standard Token Filter, Lower Case Token Filter, and Stop Token Filter.

**Standard Tokenizer** 是基于语法(英语)做分词的, 并且把标点符号比如上面的 `&` 和 `!` 去掉, **Lower Case Token Filter** 则是把所有的单词都统一成小写, 除此以外, standard analyzer 还用到了 **Stop Token Filter**, 没说设置了哪些停词, 所以, 这里看不到其作用, 不过可以肯定的是, 单词 `the` 不是停词, 其实, 我们可以设置单词 `the` 为 **stopwords**, 因为其实对于索引而言, 这个词的并没有什么意义, 可以去掉.  最后, **term_freq** 表示每个词出现的次数.


基于如上的结果来做一些搜索, 先试试这个:

```javascript
curl -XGET 'http://localhost:9200/blog/post/_search?pretty=true' -d '
{
  "query" : {
    "match" : {
      "title" : "dog"
    }
  }
}'
```

输出的结果, 居然是.......

```javascript
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 0,
    "max_score" : null,
    "hits" : [ ]
  }
}
```

什么鬼, 居然搜不到???  其实, 仔细看就会发现其实也不奇怪, 我们搜索的关键词是 **dog**, 但是 title 的倒排索引索引中的词只有 `[cats, dogs, good, the]` 这几个词, 注意是 **dogs** 不是我们要搜索的 **dog**

好吧, 瞬间觉得好不智能是伐 = . =

下面换一个 snowball analyzer 再试试看, 不过要先删掉当前索引, 再重新跑一次(**mappings 每次更改都要修改 reindex 才能生效**)

```ruby
curl -XDELETE 'http://localhost:9200/blog'
```

```javascript
curl -XPUT 'http://localhost:9200/blog/' -d '
{
  "mappings": {
     "post": {
        "properties": {
           "title": {
              "type": "string",
              "analyzer": "snowball",
              "term_vector": "yes"
           }
        }
     }
  }
}'

```

还是存入同样的数据,

```javascript
curl -XPUT 'http://localhost:9200/blog/post/1?pretty=true' -d '
{
    "title": "the Good Cats & the Good Dogs!"
}
'
```

再次, 使用 **termvector** 来查看 title 这个 field 的数据是如何存储的

```javascript
curl -XGET 'http://localhost:9200/blog/post/1/_termvector?fields=title&pretty=true'

{
  "_index" : "blog",
  "_type" : "post",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "took" : 1,
  "term_vectors" : {
    "title" : {
      "field_statistics" : {
        "sum_doc_freq" : 3,
        "doc_count" : 1,
        "sum_ttf" : 4
      },
      "terms" : {
        "cat" : {
          "term_freq" : 1
        },
        "dog" : {
          "term_freq" : 1
        },
        "good" : {
          "term_freq" : 2
        }
      }
    }
  }
}

```

可以看到, 这一次, 存入的词语只有 `[cat, dog, good]`, [官方描述 Snowball Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-snowball-analyzer.html)

> An analyzer of type snowball that uses the standard tokenizer, with standard filter, lowercase filter, stop filter, and [snowball filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-snowball-tokenfilter.html).

大部分跟 standard analyzer 差不多, 从结果看出区别, snowball 把 `the` 设置为停词了, 然后还多了个 snowball filter. 没有详细深挖这个 filter, 不过大意应该是提取词干, 比如 computing 和 computed 的词干 comput, 或者上面的 dogs, cats 变为 dog, cat.

现在再去搜索 `dog`, 肯定是可以命中的.

那么问题来了, 如果我搜索的是 dogs 呢? 会不会不中? 答案是, 能够命中的, 因为这里搜索 dogs, 会先把搜索的词同样的处理为 dog 再去匹配.

```javascript
curl -XGET 'http://localhost:9200/blog/post/_search?pretty=true' -d '
{
  "query" : {
    "match" : {
      "title" : "dogs"
    }
  }
}'
```

```javascript
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.2169777,
    "hits" : [ {
      "_index" : "blog",
      "_type" : "post",
      "_id" : "1",
      "_score" : 0.2169777,
      "_source" : {
        "title" : "the Good Cats & the Good Dogs!"
      }
    } ]
  }
}

```

--------------

以上, 大概说了一些默认的 Analyzers, 接下来, 看下怎么自定义 Analyzers 并运用到 mappings 中.


官方给出了一个自定义 analyzer 语法例子如下:

```yml
index :
    analysis :
        analyzer :
            myAnalyzer2 :
                type : custom
                tokenizer : myTokenizer1
                filter : [myTokenFilter1, myTokenFilter2]
                char_filter : [my_html]
                position_increment_gap: 256
        tokenizer :
            myTokenizer1 :
                type : standard
                max_token_length : 900
        filter :
            myTokenFilter1 :
                type : stop
                stopwords : [stop1, stop2, stop3, stop4]
            myTokenFilter2 :
                type : length
                min : 0
                max : 2000
        char_filter :
              my_html :
                type : html_strip
                escaped_tags : [xxx, yyy]
                read_ahead : 1024
```

接下来, 试下写一个自己的.

上面的第一个栗子中, 可以看到, 当存入的是 dogs, 搜索 dog 都不能命中, 更不用说搜索 `do` `go` 什么的.

很多时候我们一些电商网站, 输入一两个字符, 就会给出一些提示选项给我们, 如图:

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f6brqidftjj20z40hcn14.jpg)

这种 **Autocomplete** 的功能需要把字符串分的粒度很细, 这时候我们就可以用到 [Ngrams for Partial Matching](https://www.elastic.co/guide/en/elasticsearch/guide/current/_ngrams_for_partial_matching.html)

写一个用于 **Autocomplete** 的 **Analyzer**.

```ruby

curl -XDELETE 'http://localhost:9200/blog'

```

```javascript
curl -XPUT 'http://localhost:9200/blog/' -d '
{
  "settings" : {
    "analysis" : {
      "analyzer" : {
        "autocomplete_analyzer": {
          "type" : "custom",
          "char_filter" : ["replace_ampersands"],
          "tokenizer" : "my_edgeNGram_tokenizer",
          "filter" : ["lowercase", "my_stop"]
        }
      },
      "char_filter" : {
                     "replace_ampersands" : {
                         "type" : "mapping",
                         "mappings" : ["&=>and"]
                     }
      },
      "tokenizer" : {
        "my_edgeNGram_tokenizer" : {
          "type" : "edgeNGram",
          "min_gram" : "2",
          "max_gram" : "8",
          "token_chars": [ "letter", "digit" ]
        }
      },
      "filter" : {
          "my_stop" : {
              "type" :      "stop",
              "stopwords" : ["the"]
          }
      }
    }
  },
  "mappings": {
     "post": {
        "properties": {
           "title": {
              "type": "string",
              "analyzer": "autocomplete_analyzer",
              "term_vector": "yes"
           }
        }
     }
  }
}
'

```

其中, **my_stop** 这个 filter 和 **replace_ampersands** 这个 char_filter 其实可以不用, 只是拿来示范一下,
my_stop 是把单词 `the` 去掉, 这里细分以后, 就会很奇怪的**有 th, 但是没有 the**, 因为 the 被停词去掉了, 好吧, 在实际中不会这么去用 nGram,

`replace_ampersands` 这个 char_filter 是在分词前预处理字符串, **把 & 变成 and**.

值得注意的是, 这里用的是 **edgeNGram**, 而不是 **nGram**, 从名字可以猜出来, 结果看后面


存入数据,

```javascript
curl -XPUT 'http://localhost:9200/blog/post/1?pretty=true' -d '
{
    "title": "the Cats & the Dogs!"
}
'
```

Again, 使用 **termvector** 来查看此时 title 是如何存储的

```javascript
curl -XGET 'http://localhost:9200/blog/post/1/_termvector?fields=title&pretty=true'

{
  "_index" : "blog",
  "_type" : "post",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "took" : 1,
  "term_vectors" : {
    "title" : {
      "field_statistics" : {
        "sum_doc_freq" : 9,
        "doc_count" : 1,
        "sum_ttf" : 10
      },
      "terms" : {
        "an" : {
          "term_freq" : 1
        },
        "and" : {
          "term_freq" : 1
        },
        "ca" : {
          "term_freq" : 1
        },
        "cat" : {
          "term_freq" : 1
        },
        "cats" : {
          "term_freq" : 1
        },
        "do" : {
          "term_freq" : 1
        },
        "dog" : {
          "term_freq" : 1
        },
        "dogs" : {
          "term_freq" : 1
        },
        "th" : {
          "term_freq" : 2
        }
      }
    }
  }
}

```

倒排索引的粒度变细了, 这样, 在输入前面两个字符 `th` 就可以命中匹配. edgeNGram 是仅从词头开始分词, 这里如果用的是 nGram 的话, 那么粒度会更细, 会多出一些在 **edgeNGram** 中没的, 比如, `ats`, `ogs` 等等.
min_gram 和 max_gram 都还比较好理解, **token_chars** 是指分割点, 比如这里加了 letter 和 digit, 因为空格不属于这两样, 那么**空格就会被当成分割点**, 倒排索引里也就看不到有空格. 如果要保留空格, 也就是留下 `the c` 和 `the ca` ...这些分词的话, 就得再加上 **whitespace** 了.



```javascript
curl -XGET 'http://localhost:9200/blog/post/_search?pretty=true' -d '
{
  "query" : {
    "match" : {
      "title" : "th"
    }
  }
}'
```

```javascript
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.13561106,
    "hits" : [ {
      "_index" : "blog",
      "_type" : "post",
      "_id" : "1",
      "_score" : 0.13561106,
      "_source" : {
        "title" : "the Cats & the Dogs!"
      }
    } ]
  }
}
```

------------

最后, 如果只是想精确的存储值而不被分析的话, 可以用 **"index": "not_analyzed"**

比如 `United Kingdom` 就是 `United Kingdom`, 不想被拆成 `United`, `Kingdom`什么的, 这种时候就可以用上了, not_analyzed 的效率会高一些.

long, double, date 这些类型的数据是永远不会被分析的. 要么是 `"index" : "no"`,  或者 `"index" : "not_analyzed"`

Last but not least, 如果想知道具体怎么计算出来的匹配得分, 还可以看看 [Explain API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-explain.html)

------------

相关阅读:
[https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)

[How search and index works (Ruby 语言描述)](https://ruby-china.org/topics/23447)

[An Introduction to Ngrams in Elasticsearch](https://qbox.io/blog/an-introduction-to-ngrams-in-elasticsearch)