---
title:      ElasticSearch
---
# ElasticSearch
## 基础
### 为了搜索
#### 安装和运行
```
./bin/elasticsearch
```
测试 Elasticsearch 是否启动成功

```
curl "http://localhost:9200/?pretty"
```
##### 安装Sense
下载kibana

```
 https://download.elastic.co/elastic/sense/sense-latest.tar.gz
```
在 Kibana 目录下运行下面的命令，下载并安装 Sense app

```
./bin/kibana plugin --install elastic/sense
```
win：
```
bin\kibana.bat plugin --install elastic/sense
```

启动 Kibana

```
./bin/kibana
```
win:
```
bin\kibana.bat
```

访问：

```
http://localhost:5601
```
#### 和ES交互
计算集群中文档的数量：

```
GET /_count
{
    "query": {
        "match_all": {}
    }
}
```
#### 索引
- 一个 Elasticsearch 集群可以包含多个索引
- 相应的每个索引可以包含多个类型
- 这些不同的类型存储着多个文档
- 每个文档又有多个属性 

索引：
- 一个索引类似于传统关系数据库中的一个数据库，是一个存储关系型文档的地方。 
- 索引一个文档 就是存储一个文档到一个索引（名词）中以便它可以被检索和查询到。
- 关系型数据库通过增加一个索引比如一个B树（B-tree）索引到指定的列上，以便提升数据检索速度。
- 一个文档中的每一个属性都是被索引的（有一个倒排索引）和可搜索的。一个没有倒排索引的属性是不能被搜索到的。

**索引一个文档：**

```
PUT /megacorp/employee/1
{
  "first_name": "John",
  "last_name": "Smith",
  "age": 25,
  "about": "I love to go rock climbing",
  "interests": [
    "sports",
    "music"
  ]
}
```
- megacorp：索引名称
- employee：类型名称
- 1：特定雇员的ID

#### 检索文档
##### 增
```
PUT /megacorp/employee/1
{
  "first_name": "John",
  "last_name": "Smith",
  "age": 25,
  "about": "I love to go rock climbing",
  "interests": [
    "sports",
    "music"
  ]
}
```
##### 删
```
DELETE /megacorp/employee/1
```
##### 改
```
PUT /megacorp/employee/1
{
  "first_name": "John",
  "last_name": "Smith",
  "age": 25,
  "about": "I love to go rock climbing",
  "interests": [
    "sports",
    "music"
  ]
}
```
##### 查
```
GET /megacorp/employee/1
```

```
HEAD /megacorp/employee/1
```
#### 轻量搜索
搜索所有雇员：

```
GET /megacorp/employee/_search
```
**高亮搜索**

```
GET /megacorp/employee/_search?q=last_name:Smith
```
#### 使用查询表达式搜索

```
POST /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "Smith"
    }
  }
}
```

#### 短语搜索
- 精确匹配一系列单词或者短语 。
- 比如， 我们想执行这样一个查询，仅匹配同时包含 “rock” 和 “climbing” 
- 并且 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。

```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}

```
#### 高亮搜索

```
GET /megacorp/employee/_search
{
  "query": {
    "match_phrase": {
      "about": "rock climbing"
    }
  },
  "highlight": {
    "fields": {
      "about": {}
    }
  }
}
```

```
"highlight": {
          "about": [
            "I love to go <em>rock</em> <em>climbing</em>"
          ]
        }
```
#### 分析

```
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```

```
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```
查询特定兴趣爱好员工的平均年龄：

```
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}

```

```
"all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```
### 集群内的原理
如果我们启动了一个单独的节点，里面不包含任何的数据和 索引，那我们的集群看起来就是一个包含空内容节点的集群。
![17](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/17.png)
一个运行中的 Elasticsearch 实例称为一个 节点，而集群是由一个或者多个拥有相同 cluster.name 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

一个节点被选举成为 主 节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

作为用户，我们可以将请求发送到 集群中的任何节点 ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。

#### 集群健康

```
GET _cluster/health
```

```
{
  "cluster_name": "elasticsearch",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 5,
  "active_shards": 5,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 5,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50
}
```
status的三种颜色含义如下：

green

所有的主分片和副本分片都正常运行。

yellow

所有的主分片都正常运行，但不是所有的副本分片都正常运行。

red

有主分片没能正常运行。

集群的健康状况为 yellow 则表示全部 主 分片都正常运行（集群可以正常服务所有请求），但是 副本 分片没有全部处在正常状态。 实际上，所有3个副本分片都是 unassigned —— 它们都没有被分配到任何节点。 在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据。
#### 添加索引
索引实际上是指向一个或者多个物理 分片 的 逻辑命名空间 。

一个 分片 是一个底层的 工作单元 ，它仅保存了 全部数据中的一部分。一个分片是一个 Lucene 的实例，以及它本身就是一个完整的搜索引擎。我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互。

分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。 当你的集群规模扩大或者缩小时， Elasticsearch 会自动的在各节点中迁移分片，使得数据仍然均匀分布在集群里。

一个分片可以是 主 分片或者 副本 分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。

> 技术上来说，一个主分片最大能够存储 Integer.MAX_VALUE - 128 个文档，但是实际最大值还需要参考你的使用场景：包括你使用的硬件， 文档的大小和复杂程度，索引和查询文档的方式以及你期望的响应时长。

一个副本分片只是一个主分片的拷贝。 副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。

在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。

索引在默认情况下会被分配5个主分片， 但是为了演示目的，我们将分配3个主分片和一份副本（每个主分片拥有一个副本分片）：

```
PUT /blogs
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
```
当第二个节点加入到集群后，3个 副本分片 将会分配到这个节点上——每个主分片对应一个副本分片。 这意味着当集群内任何一个节点出现问题时，我们的数据都完好无损。

所有新近被索引的文档都将会保存在主分片上，然后被并行的复制到对应的副本分片上。这就保证了我们既可以从主分片又可以从副本分片上获得文档。

拥有6个分片（3个主分片和3个副本分片）的索引可以最大扩容到6个节点，每个节点上存在一个分片，并且每个分片拥有所在节点的全部资源。

如果我们想要扩容超过6个节点怎么办呢？

主分片的数目在索引创建时 就已经确定了下来。实际上，这个数目定义了这个索引能够 存储 的最大数据量。（一个主分片最大能够存储 Integer.MAX_VALUE（21 4748 3647） - 128 ）

但是，读操作——搜索和返回数据——可以同时被主分片 或 副本分片所处理，所以当你拥有越多的副本分片时，也将拥有越高的吞吐量。

在运行中的集群上是可以动态调整副本分片数目的 ，我们可以按需伸缩集群。让我们把副本数从默认的 1 增加到 2 ：

```
PUT /blogs/_settings
{
   "number_of_replicas" : 2
}
```
#### 应对故障
如果我们关闭第一个节点

我们关闭的节点是一个主节点。而集群必须拥有一个主节点来保证正常工作，所以发生的第一件事情就是选举一个新的主节点： Node 2 。

在我们关闭 Node 1 的同时也失去了主分片 1 和 2 ，并且在缺失主分片的时候索引也不能正常工作。 如果此时来检查集群的状况，我们看到的状态将会为 red ：不是所有主分片都在正常工作。

幸运的是，在其它节点上存在着这两个主分片的完整副本， 所以新的主节点立即将这些分片在 Node 2 和 Node 3 上对应的副本分片提升为主分片， 此时集群的状态将会为 yellow 。 这个提升主分片的过程是瞬间发生的，如同按下一个开关一般。

### 数据输入和输出
#### 文档元数据
三个必须的元数据元素如下：
- _index : 文档在哪存放
- _type ：文档表示的对象类别
- _id ： 文档唯一标识

##### _index
一个 索引 应该是因共同的特性被分组到一起的文档集合。例如，你可能存储所有的产品在索引 products 中，而存储所有销售的交易到索引 sales 中。

> 实际上，在 Elasticsearch 中，我们的数据是被存储和索引在 分片 中，而一个索引仅仅是逻辑上的命名空间， 这个命名空间由一个或者多个分片组合在一起。 然而，这是一个内部细节，我们的应用程序根本不应该关心分片，对于应用程序而言，只需知道文档位于一个 索引 内。 Elasticsearch 会处理所有的细节。

**索引名，名字必须小写，不能以下划线开头，不能包含逗号。**

##### _type
**_type 命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符**

##### _id

#### 索引文档
##### 使用自定义ID

```
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}
```
##### 自生成ID

```
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```
##### 取回一个文档
###### 返回文档一部分

```
GET /website/blog/123?_source=title,text
```

```
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```
##### 检查文档是否存在

```
HEAD megacorp/employee/1
```

```
200 - OK
```

```
HEAD megacorp/employee/5
```

```
404 - Not Found
```
##### 更新整个文档

```
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text": "I am starting to get the hang of this...",
  "date": "2014/01/02"
}
```

```
{
  "_index": "website",
  "_type": "blog",
  "_id": "123",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```
version:2。代表被更新了

在内部，Elasticsearch 已将旧文档标记为已删除，并增加一个全新的文档。 尽管你不能再对旧版本的文档进行访问，但它并不会立即消失。当继续索引更多的数据，Elasticsearch 会在后台清理这些已删除文档。
###### update API
- 从旧文档构建 JSON
- 更改该 JSON
- 删除旧文档
- 索引一个新文档
##### 创建新文档


### 搜索
文档中的每个字段都将被索引并且可以被查询 。
搜索（search） 可以做到：
- 在类似于 gender 或者 age 这样的字段 上使用结构化查询，join_date 这样的字段上使用排序，就像SQL的结构化查询一样。
- 全文检索，找出所有匹配关键字的文档并按照相关性（relevance） 排序后返回结果。
- 以上二者兼而有之。
#### 空搜索

```
GET /_search
```

```
{
   "hits" : {
      "total" :       14,
      "hits" : [
        {
          "_index":   "us",
          "_type":    "tweet",
          "_id":      "7",
          "_score":   1,
          "_source": {
             "date":    "2014-09-17",
             "name":    "John Smith",
             "tweet":   "The Query DSL is really powerful and flexible",
             "user_id": 2
          }
       },
        ... 9 RESULTS REMOVED ...
      ],
      "max_score" :   1
   },
   "took" :           4,
   "_shards" : {
      "failed" :      0,
      "successful" :  10,
      "total" :       10
   },
   "timed_out" :      false
}
```
##### hits
包含 total 字段来表示匹配到的文档总数，并且一个 hits 数组包含所查询结果的前十个文档
##### _score
衡量了文档与查询的匹配程度，返回的文档是按照 _score 降序排列的。
##### took
took 值告诉我们执行整个搜索请求耗费了多少毫秒。
##### timeout 
timed_out 值告诉我们查询是否超时。
如果低响应时间比完成结果更重要，你可以指定 timeout 为 10 或者 10ms（10毫秒），或者 1s（1秒）：

```
GET /_search?timeout=10ms
```
> 应当注意的是 timeout 不是停止执行查询，它仅仅是告知正在协调的节点返回到目前为止收集的结果并且关闭连接。在后台，其他的分片可能仍在执行查询即使是结果已经被发送了。
使用超时是因为 SLA(服务等级协议)对你是很重要的，而不是因为想去中止长时间运行的查询。

#### 多索引，多类型
/索引/类型/_search

```
/_search
在所有的索引中搜索所有的类型
/gb/_search
在 gb 索引中搜索所有的类型
/gb,us/_search
在 gb 和 us 索引中搜索所有的文档
/g*,u*/_search
在任何以 g 或者 u 开头的索引中搜索所有的类型
/gb/user/_search
在 gb 索引中搜索 user 类型
/gb,us/user,tweet/_search
在 gb 和 us 索引中搜索 user 和 tweet 类型
/_all/user,tweet/_search
在所有的索引中搜索 user 和 tweet 类型
```
#### 分页
##### from 和 size 参数
size：显示应该返回的结果数量，默认是 10

from：显示应该跳过的初始结果数量，默认是 0
##### 栗子
如果每页展示 5 条结果，可以用下面方式请求得到 1 到 3 页的结果：

```
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```
> 请记住一个请求经常跨越多个分片，每个分片都产生自己的排序结果，这些结果需要进行集中排序以保证整体顺序是正确的

#### 轻量搜索
##### 两种形式的 搜索 API
###### “轻量的” 查询字符串
- "+" 前缀表示必须与查询条件匹配
- "-" 前缀表示一定不与查询条件匹配
-  没有 "+" 或者 "-" 的所有其他条件都是可选的
-  ":" 包含

**例子：**

```
GET /_all/tweet/_search?q=tweet:elasticsearch
```

tweet 类型中 tweet 字段包含 elasticsearch 单词的所有文档

```
GET /_all/tweet/_search?q=+name:john +tweet:mary
```

name 字段中包含 john 并且在 tweet 字段中包含 mary 的文档

```
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```
字符串参数URL编码
###### _all

```
GET /_search?q=mary
```
当索引一个文档的时候，Elasticsearch取出所有字段的值拼接成一个大的字符串，作为 _all 字段进行索引。

```
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}
```
这就好似增加了一个名叫 _all 的额外字段：

```
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
```
###### 更复杂的查询
**栗子：**
- name 字段中包含 mary 或者 john
- date 值大于 2014-09-10
- _all_ 字段包含 aggregations 或者 geo

```
+name:(mary john) +date:>2014-09-10 +(aggregations geo)
```
### 映射和分析

```
GET /索引/_mapping/类型
```

```
GET /feed_sit/_mapping/feed_sit
```

```
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

#### 精确值和全文
Elasticsearch 中的数据可以概括的分为两类：精确值和全文。

#### 倒排索引

- Elasticsearch使用一种称为倒排索引的结构，它适用于快速的全文搜索。
- 一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。
- '+' 前缀表明这个词必须存在。
- 分词和标准化的过程称为 分析 

#### 分析和分析器
分析 包含下面的过程：
- 首先，将一块文本分成适合于倒排索引的独立的 词条 
- 之后，将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 recall

###### 字符过滤器

首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 `and`。

###### 分词器

其次，字符串被 分词器分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

###### Token 过滤器

最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a`， `and`， `the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）。

##### 内置分析器
###### 标准分析器

根据 Unicode 联盟 定义的 单词边界 划分文本。删除绝大部分标点。最后，将词条小写。

###### 简单分析器
简单分析器在任何不是字母的地方分隔文本，将词条小写。

###### 语言分析器
特定语言分析器可用于很多语言。英语分析器附带了一组英语无用词（常用单词，例如 and 或者 the，它们对相关性没有多少影响），它们会被删除。由于理解英语语法的规则，这个分词器可以提取英语单词的 词干 。

##### 测试分析器

```
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
```
##### 指定分析器
当Elasticsearch在你的文档中检测到一个新的字符串域 ，它会自动设置其为一个全文 字符串 域，使用 标准 分析器对它进行分析。

你想使用一个不同的分析器，适用于你的数据使用的语言。有时候你想要一个字符串域就是一个字符串域--不使用分析，直接索引你传入的精确值，例如用户ID或者一个内部的状态域或标签。

要做到这一点，我们必须手动指定这些域的映射。

#### 映射
Elasticsearch需要知道每个域中数据的类型。这个信息包含在映射中。

##### 核心简单域类型
Elasticsearch 支持 如下简单域类型：

- 字符串: string
- 整数 : byte, short, integer, long
- 浮点数: float, double
- 布尔型: boolean
- 日期: date

当你索引一个包含新域的文档--之前未曾出现-- Elasticsearch会使用动态映射，通过JSON中基本数据类型，尝试猜测域类型。

> 这意味着如果你通过引号("123")索引一个数字，它会被映射为string类型，而不是long。但是，如果这个域已经映射为long，那么Elasticsearch 会尝试将这个字符串转化为long，如果无法转化，则抛出一个异常。

##### 查看映射

```
GET /gb/_mapping/tweet
```

```
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "strict_date_optional_time||epoch_millis"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```
###### 自定义域映射
- 域最重要的属性是 type 。对于不是 string 的域，你一般只需要设置 type ：
- type = string(text)
    - index : 怎样索引字符串
        - analyzed : 首先分析字符串，然后索引它
        - 索引这个域，所以它能够被搜索，但索引的是精确值。不会对它进行分析
        - 不索引这个域。这个域不会被搜索到。
    - analyzer:
        - standard、whitespace、simple、english


```
{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}
```

```
{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}
```


###### 更新映射

```
DELETE /gb
```

```
PUT /gb
{
  "mappings": {
    "tweet": {
      "properties": {
        "tweet": {
          "type": "text",
          "analyzer": "english"
        },
        "date": {
          "type": "date"
        },
        "name": {
          "type": "text"
        },
        "user_id": {
          "type": "long"
        }
      }
    }
  }
}
```
增加一个新的名为 tag 的 not_analyzed 的文本域：

```
PUT /gb/_mapping/tweet
{
  "properties" : {
    "tag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}
```
不需要再次列出所有已存在的域，因为无论如何我们都无法改变它们。新域已经被合并到存在的映射中。



### 请求体查询
#### 空查询

```
POST /index/type1,type2/_search
{}
```
#### from 和 size 参数


```
POST /_search
{
  "from": 30,
  "size": 10
}
```
#### 查询表达式
##### 查询语句的结构

```
POST /_search
{
    "query": {
        "match_all": {}
    }
}
```

```
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```
##### 合并查询语句
一条复合语句可以合并任何其它查询语句，包括复合语句。这就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。


```
POST /feed_sit/feed_sit/_search?pretty=true
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "info_title": "区块链"
        }
      },
      "must_not": {
        "match": {
          "name": "mary"
        }
      },
      "should": {
        "match": {
          "tweet": "full text"
        }
      },
      "filter": {
        "range": {
          "age": {
            "gt": 30
          }
        }
      }
    }
  }
}
```
嵌套

```
POST /feed_sit/feed_sit/_search?pretty=true
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "email": "business opportunity"
        }
      },
      "should": [
        {
          "match": {
            "starred": true
          }
        },
        {
          "bool": {
            "must": {
              "match": {
                "folder": "inbox"
              }
            },
            "must_not": {
              "match": {
                "spam": true
              }
            }
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```
#### 查询与过滤
> 从 Elasticsearch2.0开始，过滤（filters）已经从技术上被排除了，同时所有的查询（queries）拥有变成不评分查询的能力。

> 我们用 "filter" 这个词表示不评分、只过滤情况下的查询。你可以把 "filter" 、 "filtering query" 和 "non-scoring query" 这几个词视为相同的。

> 过滤（filtering）的目标是减少那些需要通过评分查询（scoring queries）进行检查的文档。 

#### 最重要的查询
##### match_all查询
match_all 查询简单的匹配所有文档。在没有指定查询方式时，它是默认的查询：

```
{ "match_all": {}}
```
##### match查询

```
{ "match": { "tweet": "About Search" }}
```
> 对于精确值的查询，你可能需要使用 filter 语句来取代query，因为 filter 将会被缓存

###### multi_match查询
在 content_keywords、title_keywords字段中，查询“区块链”关键字的信息
```
POST /feed_sit/feed_sit/_search?pretty=true
{
  "query": {
    "multi_match": {
      "query": "区块链",
      "fields": ["content_keywords","title_keywords"]
    }
  }
}
```

###### range查询

```
POST /feed_sit/feed_sit/_search?pretty=true
{
  "query": {
    "range": {
      "info_push_time": {
        "gte": "2018-03-05 16:51:04",
        "lt": "2018-03-08 16:51:04"
      }
    }
  }
}
```
###### term查询
term 查询被用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 not_analyzed 的字符串。

```
POST /feed_sit/feed_sit/_search?pretty=true
{
  "query": {
    "term": {
      "info_push_time": {
        "value": "2018-03-05 20:36:01"
      }
    }
  }
}
```
###### terms查询
允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件。

```
POST /feed_sit/feed_sit/_search?pretty=true
{
  "query": {
    "terms": {
      "FIELD": [
        "VALUE1",
        "VALUE2"
      ]
    }
  }
}
```
###### exists 查询和 missing 查询
用来查找有无该字段的文档

```
POST /feed_sit/feed_sit/_search?pretty=true
{
  "query": {
    "exists":{
      "field":"info_title"
    }
  }
}
```

#### 组合多查询
需要在多个字段上查询多种多样的文本，并且根据一系列的标准来过滤。

可以用 bool 查询来实现需求。这种查询将多查询组合在一起，成为用户自己想要的布尔查询。

相关性得分是如何组合的。每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来，bool查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。

参数：
> must
文档 必须 匹配这些条件才能被包含进来。

> must_not
文档 必须不 匹配这些条件才能被包含进来。

> should
如果满足这些语句中的任意语句，将增加 _score ，否则，无任何影响。它们主要用于修正每个文档的相关性得分。

> filter
必须 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

**栗子：**

下面的查询用于查找：

title 字段匹配 how to make millions

并且tag 字段不被标识为 spam 的文档

那些被标识为 starred或在2014之后的文档，将比另外那些文档拥有更高的排名

如果 两者都满足，那么它排名将更高：

```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```
> 如果没有 must 语句，那么至少需要能够匹配其中的一条 should 语句。但，如果存在至少一条must语句，则对should语句的匹配没有要求。

##### 过滤器

```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }} 
        }
    }
}
```
如果你需要通过多个不同的标准来过滤你的文档，bool查询本身也可以被用做不评分的查询。

```
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "bool": { 
              "must": [
                  { "range": { "date": { "gte": "2014-01-01" }}},
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```
##### constant_score查询
它被经常用于你只需要执行一个filter而没有其它查询（例如，评分查询）的情况下。

可以使用它来取代只有filter语句的bool查询。在性能上是完全相同的，但对于提高查询简洁性和清晰度有很大帮助。

```
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" } 
        }
    }
}
```
#### 验证查询
### 排序和相关性
#### 排序
在 Elasticsearch 中， 相关性得分 由一个浮点数进行表示，并在搜索结果中通过_score 参数返回， 默认排序是 _score 降序。

```
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```
此时返回的评分没有意义，可以采用下面的写法：

```
GET /_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```
评分默认为：1

#### 按照字段的值进行排序

```
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : { "term" : { "user_id" : 1 }}
        }
    },
    "sort": { "date": { "order": "desc" }}
}
```
计算 _score 的花销巨大，通常仅用于排序； 我们并不根据相关性排序，所以记录_score 是没有意义的。如果无论如何你都要计算 _score，你可以将track_scores参数设置为true 。
 
#### 多级排序
结合使用 date 和 _score进行查询，匹配的结果首先按照日期排序，然后按照相关性排序。

结果首先按第一个条件排序，仅当结果集的第一个sort值完全相同时才会按照第二个条件进行排序，以此类推。

```
GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
    "sort": [
        { "date":   { "order": "desc" }},
        { "_score": { "order": "desc" }}
    ]
}
```
#### 字符段排序与多字段
##### 什么是相关性
默认情况下，返回结果是按相关性倒序排列的。

fuzzy 查询会计算与关键词的拼写相似程度，terms查询会计算找到的内容与关键词组成部分匹配的百分比。

Elasticsearch 的相似度算法 被定义为检索词频率/反向文档频率， TF/IDF ，包括以下内容：
- 检索词频率：

    检索词在该字段出现的频率？出现频率越高，相关性也越高。 字段中出现过 5 次要比只出现过 1 次的相关性高。
- 反向文档频率：
    
    每个检索词在索引中出现的频率？频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。
- 字段长度准则：

    字段的长度是多少？长度越长，相关性越低。 检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段权重更大。

如果多条查询子句被合并为一条复合查询语句 ，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。

##### 理解评分标准

```
GET /_search?explain 
{
   "query"   : { "match" : { "tweet" : "honeymoon" }}
}
```

```
"_explanation": { //honeymoon 相关性评分计算的总结
   "description": "weight(tweet:honeymoon in 0)
                  [PerFieldSimilarity], result of:",
   "value":       0.076713204,
   "details": [
      {
         "description": "fieldWeight in 0, product of:",
         "value":       0.076713204,
         "details": [
            {  //检索词频率
               "description": "tf(freq=1.0), with freq of:",
               "value":       1,
               "details": [
                  {
                     "description": "termFreq=1.0",
                     "value":       1
                  }
               ]
            },
            { //反向文档频率
               "description": "idf(docFreq=1, maxDocs=1)",
               "value":       0.30685282
            },
            { //字段长度准则
               "description": "fieldNorm(doc=0)",
               "value":        0.25,
            }
         ]
      }
   ]
}
```
> 输出 explain 结果代价是十分昂贵的，它只能用作调试工具。千万不要用于生产环境。

检索词频率:
> 检索词 `honeymoon` 在这个文档的 `tweet` 字段中的出现次数。

反向文档频率:
> 检索词 `honeymoon` 在索引上所有文档的 `tweet` 字段中出现的次数。

字段长度准则:
> 在这个文档中， `tweet` 字段内容的长度 -- 内容越长，值越小。

## 深入搜索
### 结构化搜索
#### 精确值查找
当进行精确值查找时， 我们会使用过滤器（filters）。过滤器很重要，因为它们执行速度非常快，不会计算相关度（直接跳过了整个评分阶段）而且很容易被缓存。请尽可能多的使用过滤式查询。

##### term查询数字

```
{
    "term" : {
        "price" : 20
    }
}
```
不评分：

```
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : { 
            "filter" : {
                "term" : { 
                    "price" : 20
                }
            }
        }
    }
}
```
##### term查询文本

```
SELECT product
FROM   products
WHERE  productID = "XHDK-A-1293-#fJ3"
```

```
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "productID" : "XHDK-A-1293-#fJ3"
                }
            }
        }
    }
}
```

```
GET /my_store/_analyze
{
  "field": "productID",
  "text": "XHDK-A-1293-#fJ3"
}

```

```
{
  "tokens" : [ {
    "token" :        "xhdk",
    "start_offset" : 0,
    "end_offset" :   4,
    "type" :         "<ALPHANUM>",
    "position" :     1
  }, {
    "token" :        "a",
    "start_offset" : 5,
    "end_offset" :   6,
    "type" :         "<ALPHANUM>",
    "position" :     2
  }, {
    "token" :        "1293",
    "start_offset" : 7,
    "end_offset" :   11,
    "type" :         "<NUM>",
    "position" :     3
  }, {
    "token" :        "fj3",
    "start_offset" : 13,
    "end_offset" :   16,
    "type" :         "<ALPHANUM>",
    "position" :     4
  } ]
}
```
- Elasticsearch 用 4 个不同的 token 而不是单个 token 来表示这个 UPC 。
- 所有字母都是小写的。
- 所有字母都是小写的。
#### 组合过滤器
##### 布尔过滤器

```
{
   "bool" : {
      "must" :     [],
      "should" :   [],
      "must_not" : [],
   }
}
```
must

    所有的语句都 必须（must） 匹配，与 AND 等价
must_not

    所有的语句都 不能（must not） 匹配，与 NOT 等价。
should

    至少有一个语句要匹配，与 OR 等价。

**栗子：**

```
SELECT product
FROM   products
WHERE  (price = 20 OR productID = "XHDK-A-1293-#fJ3")
  AND  (price != 30)
```

```
GET /my_store/products/_search
{
   "query" : {
      "filtered" : { 
         "filter" : {
            "bool" : {
              "should" : [
                 { "term" : {"price" : 20}}, 
                 { "term" : {"productID" : "XHDK-A-1293-#fJ3"}} 
              ],
              "must_not" : {
                 "term" : {"price" : 30} 
              }
           }
         }
      }
   }
}

```
##### 嵌套布尔过滤器
**栗子：**

```
SELECT document
FROM   products
WHERE  productID = "KDKE-B-9947-#kL5"
  OR ( productID = "JODL-X-1937-#pV7"
       AND price     = 30 )
```

```
GET /my_store/products/_search
{
   "query" : {
      "filtered" : {
         "filter" : {
            "bool" : {
              "should" : [
                { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, 
                { "bool" : { 
                  "must" : [
                    { "term" : {"productID" : "JODL-X-1937-#pV7"}}, 
                    { "term" : {"price" : 30}} 
                  ]
                }}
              ]
           }
         }
      }
   }
}
```
## 管理、监控和部署
### 监控
#### Marvel监控
##### Marvel安装
need to install two components:
- An Elasticsearch plugin that collects data from each node.
- A Kibana app that provides the Marvel monitoring UI.

默认，Marvel数据存放在ES安装的目录。recommend that you store the Marvel data in a separate monitoring cluster.这样即使你的ES处于不健康的状态，Mavrel也可以对其进行监控。
### 部署
#### 重要配置的修改
其它数据库可能需要调优，但总得来说，Elasticsearch不需要。如果你遇到了性能问题，解决方法通常是更好的数据布局或者更多的节点。
###### 指定名称
Elasticsearch 默认启动的集群名字叫 elasticsearch 。简单修改成 elasticsearch_production。

elasticsearch.yml 文件中修改：
```
cluster.name: elasticsearch_production
```
最好也修改你的节点名字。

elasticsearch.yml 文件中修改：

```
node.name: elasticsearch_005_data
```
###### 路径
默认情况下， Elasticsearch会把插件、日志以及你最重要的数据放在安装目录下。这会带来不幸的事故，如果你重新安装Elasticsearch的时候不小心把安装目录覆盖了。如果你不小心，你就可能把你的全部数据删掉了。

最好的选择就是把你的数据目录配置到安装目录以外的地方，同样你也可以选择转移你的插件和日志目录。

可以更改如下：

```
path.data: /path/to/data1,/path/to/data2 

# Path to log files:
path.logs: /path/to/logs

# Path to where plugins are installed:
path.plugins: /path/to/plugins
```
