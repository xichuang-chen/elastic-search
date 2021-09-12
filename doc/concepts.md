# Arch
![img.png](assets/elastic-search-arch.png)

[参考链接](https://www.zhihu.com/question/323811022)
## Cluster
- 一个cluster由一个或多个node组成
- 一个cluster包含所有的数据
- cluster name是区分cluster的唯一标示（默认是 `elasticsearch` ）, 
  所以多个cluster名字要区分, 比如 `logging-stage` `longging-prod`等  
  Node 只能通过cluster名字加入其cluster
- [cluster](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/getting-started-concepts.html#_cluster)

## Node
- 一个 Node 并一定是一台机器，也可以一个机器上部署多个node， 当然这不是一个好的实践
- Node 和 cluster一样，都由名字唯一标示，node在启动时候会随机分配一个
- Node 包含很多 Shards

## Shards & Replicas
### 为什么分片
- 可以理解为mysql的分表
- 一个index的数据可能很大，一个node存不下，所以index存储时可以指定存到几个shards上
- 多个分片，在写入或查询的时候就可以并行操作（从各个节点中读写数据，提高吞吐量）
- 建立索引时，可以设置想要的分片数量

### 如何实现高可用
- 每个分片都有对应的副本，副本会拷贝主分片数据
- 主分片与相应的副分片不在同一node
- 每个主分片的副本可以有一个或多个
- 默认情况下，es为每个index分配5个shards，和一个replica.  
  如果你有两个node，则5个主的在一个node,另外5个备份的在一个node.

### Replicas
shards 分 primary  和  replicas


## 倒排索引
### 正排索引

| id | content | 
| ---- | ---- |  
| 1001 | Elasticsearch Docs |
| 1002 | Elasticsearch install |
| 1003 | Kibana Guides |

通过 id 的 主键索引 去搜索相应的内容。
但是对于模糊查询来说，这样的查询效率太低

### 倒排索引
![img.png](assets/倒排索引例子.png)  
根据document内容作为索引  
那么如何分词 ？  
Elasticsearch 内置了很多分词器, 中文的一般使用IK分词器 

#### Elasticsearch 内部结构什么样，如何进行查询的
![img.png](API/assets/es-search.png)  
我们输入一段文字，Elasticsearch会根据分词器对我们的那段文字进行分词
（也就是图上所看到的Ada/Allen/Sara..)，这些分词汇总起来我们叫做
`Term Dictionary`，而我们需要通过分词找到对应的记录，这些文档ID保存在
`PostingList`在Term Dictionary中的词由于是非常非常多的，所以我们会为其
进行排序，等要查找的时候就可以通过二分来查，不需要遍历整个Term Dictionary
由于Term Dictionary的词实在太多了，不可能把Term Dictionary所有的词都放在
内存中，于是Elasticsearch还抽了一层叫做Term Index，这层只存储 部分 词的
前缀，Term Index会存在内存中（检索会特别快）Term Index在内存中是以FST
（Finite State Transducers）的形式保存的，其特点是非常节省内存。  
FST有两个优点：
- 空间占用小。通过对词典中单词前缀和后缀的重复利用，压缩了存储空间
- 查询速度快。O(len(str))的查询时间复杂度。

[参考链接](https://www.zhihu.com/question/323811022/answer/981341195)

#### 倒排索引的更新
倒排索引被写入磁盘后，不可改变，永远不会修改
价值：
- 不需要锁

**如何更新倒排索引？**  
用更多的索引。 增加新的补充索引来反映新近的修改，然后查询时 综合所有索引，
这就是为啥会有segment的概念

**倒排索引如何删除？**
倒排索引不能改变。删除的话，会把需要删除的segment 文件标记为.del。这样查询时就跳过这些。这叫逻辑删除，文件还在  
倒排索引还有一个合并操作。 合并之后会删除这些 .del 

## 其他概念
### 与sql对比
使用关系型数据库的行和列存储，这相当于是把一个表现力丰富的对象塞到一个非常大的电子表格中：为了适应表结构，你必须设法将这个对象扁平化—​通常一个字段对应一列—​而且每次查询时又需要将其重新构造为对象。  
Elasticsearch 是 面向文档 的，意味着它存储整个对象或 文档。Elasticsearch 不仅存储文档，而且 索引 每个文档的内容，使之可以被检索。在 Elasticsearch 中，我们对文档进行索引、检索、排序和过滤—​而不是对行列数据。这是一种完全不同的思考数据的方式，也是 Elasticsearch 能支持复杂全文检索的原因。
#### 与sql之间的概念映射
![img.png](assets/sql-elastic.png)

## Mapping
处理数据的方式和规则方面做限制，如：某字段的数据类型、默认值、分析器、是否被索引等   
具体使用查看[mapping](./API/mapping-and-type.md)

## 分词器
elasticsearch 在写入数据和查询数据时需要使用分词器进行分词  
### 系统默认 standard 分词器
elasticsearch 系统默认使用 `standard` 分词器  
查看 elasticsearch `standard` 分词器分词效果  
```json
GET _analyze
{
  "analyzer": "standard",
  "text": "How can I learn more and more knowledge"
}
```

Response: 
```json
{
  "tokens" : [
    {
      "token" : "how",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "can",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "i",
      "start_offset" : 8,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "learn",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "more",
      "start_offset" : 16,
      "end_offset" : 20,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "and",
      "start_offset" : 21,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "more",
      "start_offset" : 25,
      "end_offset" : 29,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "knowledge",
      "start_offset" : 30,
      "end_offset" : 39,
      "type" : "<ALPHANUM>",
      "position" : 7
    }
  ]
}
```
全变小写了，所以你输入大小写都可以查询到  
还有其他一些分词器，不再概述

### 指定分词器
可以在查询或put index时指定分词器

### 自定义分词器
可以在mapping中设置
