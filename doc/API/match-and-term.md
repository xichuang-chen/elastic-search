# query 中的 match 和 term
## 区别
- 所要match的索引（如下述例子 `skill.sport` ）的值，可以被分词
- 所要term的索引的值，不能被分词

### Case1
如果所要使用的索引的值不能被分词，这种情况match 和 term 一样，如下查询
Query  
条件查询，字段中 包含这个 football 词就行, 越匹配score越高  
match 换成 terms 一样的效果
```json
GET cxc/_search   
{
  "query": {
    "match": {
      "skill.sport": "football"
    }
  }
}
```
Response  
结果中字段为 `football and basketball` 的也被检索出来了。 score 较低
```json
{
        "_index" : "cxc",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.93513453,
        "_source" : {
          "name" : "Lilei",
          "age" : 41,
          "birthDay" : "1980-10-10",
          "skill" : {
            "language" : "Englist",
            "sport" : "football"
          }
        }
      },
      {
        "_index" : "cxc",
        "_type" : "_doc",
        "_id" : "6",
        "_score" : 0.57868326,
        "_source" : {
          "name" : "Lucy",
          "age" : 22,
          "birthDay" : "1999-10-10",
          "skill" : {
            "language" : "English and Chinese",
            "sport" : "basketball and football"
          }
        }
      }
```

### Case2
如果所要使用的索引的值可以被分词
#### match
Query
```json
GET cxc/_search   
{
  "query": {
    "match": {
      "skill.sport": "football and"
    }
  }
}
```
这个查询使用 `football and` 作为查询索引，其会被分词变为 `football` 和 `and`   
从结果来看， `football and basketball`  `football` `poke ball and run` 都被检索出来了
Response
```json
{
  "_index" : "cxc",
  "_type" : "_doc",
  "_id" : "6",
  "_score" : 1.8664033,
  "_source" : {
    "name" : "Lucy",
    "age" : 22,
    "birthDay" : "1999-10-10",
    "skill" : {
      "language" : "English and Chinese",
      "sport" : "basketball and football"
    }
  }
},
{
    "_index" : "cxc",
    "_type" : "_doc",
    "_id" : "3",
    "_score" : 1.0924442,
    "_source" : {
        "name" : "Lilei",
        "age" : 41,
        "birthDay" : "1980-10-10",
        "skill" : {
            "language" : "Englist",
            "sport" : "football"
        }
  }
},
{
    "_index" : "cxc",
    "_type" : "_doc",
    "_id" : "9",
    "_score" : 0.9877362,
    "_source" : {
        "name" : "皮卡",
        "age" : 100,
        "birthDay" : "1921-10-10",
        "skill" : {
            "language" : "Japanese",
            "sport" : "poke ball and run"
        }
    }
}
```

#### term
term 不能被分词, 但是如果要用 `basketball and football` ，按说好像可以只查出来有 `basketball and football`的那条数据  
但是，查出来是空， 这又是为啥呢？
- 这可能有涉及到es存储数据时的分词器的机制， 大致上他会存储时，拆分 `basketball and football`, 而没有以`basket and football`
为索引的
- 所以， term 中一般放单个单词， 也可以放单个单词的数组，不过他们之间是 或 关系
Query
```json
GET cxc/_search   
{
  "query": {
    "match": {
      "skill.sport": "basketball and football"
    }
  }
}
```

### Case3
如果我就要精确匹配 index 为 `basketball and football` 的数据，如何做?
#### 方法1
可以使用 多条件查询 bool
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "skill.sport": "basketball"
          }
        },
        {
          "term": {
            "skill.sport": "and"
          }
        },
        {
          "term": {
            "skill.sport": "football"
          }
        }
      ]
    }
  }
}
```

#### 使用 match_phrase
他要求所有的分词必须同时出现，并且按次序挨着才算满足
Query
```json
GET cxc/_search   
{
  "query": {
    "match_phrase": {
      "skill.sport": "basketball and football"
    }
  }
}
```

## 需要知道的 es mappiing
[参考mapping](mapping-and-type.md)
