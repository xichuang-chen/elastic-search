## elasticsearch支持的数据类型

基本类型: text keyword long date double boolean ip   
还有其他一些复杂类型

### text and keyword
在es中string有两种类型， text 和 keyword
- text 可以被分词， 可以被全文检索
- keyword 不可以被分词

## mapping 
### 创建 mapping
创建索引，定义字段类型
```json
PUT mapping_text      
{
  "mappings": {
    "properties": {
      "name": {
          "type": "text",
          "index": true
      },
      "sex": {
          "type": "text",
          "index": false
        },
      "id": {
          "type": "keyword",
          "index": true
        },
      "address": {
          "type": "keyword",
          "index": false
        }
    }
  }
}
```
- index 为 mapping_test 此时还没有Put 数据
- `Properies` 定义哪些字段需要mapping  
  - `type` 表示类型
  - `index` 表示是否可以被索引


### 查询
#### Query  类型 text index: true
可以被分词就不写了
```json
GET mapping_text/_search   
{
  "query": {
    "match": {
      "name": "cxc"
    }
  }
}
```
注意： 此时不能写 name.keyword  
Response:
```json
{
        "_index" : "mapping_text",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "name" : "cxc",
          "sex" : "male",
          "id" : "1022 and 2095",
          "address" : "china xian"
        }
}
```

#### Query type: text  index: false
Query
```json
GET mapping_text/_search   
{
  "query": {
    "match": {
      "sex": "male"
    }
  }
}
```

Response 直接error  。 因为index生效，`sex` 不能被检索

#### Query type: keyword  index：false
直接error

#### Query type:keyword  index: true
```json
GET mapping_text/_search   
{
  "query": {
    "match": {
      "id": "1022"
    }
  }
}
```
查不到数据，因为存储时候，没有进行分词， 存的是  `1022 and 2095`, 没有 `1022` 的 索引  
Response
```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```

Request  
*term也可以了*
```json
GET mapping_text/_search   
{
  "query": {
    "match": {
      "id": "1022 and 2095"
    }
  }
}
```

Response
```json
{
        "_index" : "mapping_text",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.9808291,
        "_source" : {
          "name" : "cxc",
          "sex" : "male",
          "id" : "1022 and 2095",
          "address" : "china xian"
        }
      }
```