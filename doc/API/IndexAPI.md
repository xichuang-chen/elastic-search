# Index API
插入或更新index数据
参考：[Index API](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/docs-index_.html)

## 插入或更新数据

### 普通put
```json
PUT cxc/_doc/1
{
  "name": "Jim",
  "age": 30,
  "birthDay": "1991-10-10",
  "skill": {
    "language": "English",
    "sport": "basketball"
  }
}
```
插入数据 index 为 cxc， type 为 _doc， id 为 1；
如果index 为 cxc type 为 _doc 的数据中有 id=1 document， 则为更新数据

Response:
```json
{
  "_index" : "cxc",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 4,
  "result" : "updated",   // 如果为第一次创建，状态为 created
  "_shards" : {
    "total" : 2,          // 有多少shards copy (主片和副片)
    "successful" : 1,     //  有多少片成功  （应该是只要 副片拷贝）
    "failed" : 0           //  失败的片数
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

### 自动生成 id
```json
POST cxc/_doc    //  注意此处为 POST
{
  "name": "unknown",
  "age": 88,
  "birthDay": "1933-10-10",
  "skill": {
    "language": "Spanish",
    "sport": "football"
  }
}
```
Response
```json
{
  "_index" : "cxc",
  "_type" : "_doc",
  "_id" : "-psNy3sBAMrtJ44SONRr",   // 随机生成
  "_version" : 1,
  "result" : "created",     // created
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 5,
  "_primary_term" : 1
}
```


### 指定 version
为了更好的并发更新. 保证事务，不致于脏读啥的
```http request
PUT twitter/_doc/1?version=2
```