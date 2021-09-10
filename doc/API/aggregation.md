#

## Terms aggregation
[Terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/search-aggregations-bucket-terms-aggregation.html)  

Query:  
```json
GET /_search
{
    "aggs" : {
        "agg_test" : {
            "terms" : { "field" : "skill.language.keyword" }
        }
    }
}
```  
根据 `skill.language.keyword` 字段进行group， 将该字段 值相同的document group到一个bucket中
类似与sql `select COUNT(*) from table group by skill.language`
Response:  
```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "agg_test" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Chinese",
          "doc_count" : 1
        },
        {
          "key" : "Englist",
          "doc_count" : 1
        },
        {
          "key" : "Spanish",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

### 综合例子
Query
```json
GET _search
{
  "query":{
    "bool": {
      "filter": {
        "terms": { 
          "rea_categories.slug.keyword": ["selling-guide", "buying-apartments-guide", "building-guide"]   # 精确匹配
          }
      }
    }
  },
  "size": 0,       #  不要 query 回的数据
  "aggs" : {       # 第一层 aggs
      "agg_category" : {     # 随便起名
          "terms" : {        # aggs 下的terms
            "field" : "rea_categories.slug.keyword",           # 按照这个字段分组，值相同的放的一组中
            "include": ["selling-guide", "buying-apartments-guide", "building-guide"],     # 选择只包含这些值的bucket留下，其他的bucket不要，用于过滤bucket
            "size": 10
          },
          "aggs": {                  # 第二层aggs
            "agg_source": {          
              "top_hits": {         # 获取 document 的具体数据
                "_source": ["post_name"],    #  具体数据
                 "sort": [                  # 用于排序 sort bucket 内部的数据
                   {
                    "rea_fields.step_indicator.keyword": {
                      "order": "asc",
                      "missing" : "_last"
                    }
                   }
                  ],
                  "size": 10
              }
            }
          }
      }
  }
}
```
Response
```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 11,
    "successful" : 11,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 17,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "agg_category" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "building-guide",     # 由于我们在query 中加了 按照 step_indicator排序，所以bucket内部数据按照这个排序了
          "doc_count" : 7,
          "agg_source" : {
            "hits" : {
              "total" : {
                "value" : 7,
                "relation" : "eq"
              },
              "max_score" : null,
              "hits" : [
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658014",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "everything-you-need-to-know-before-deciding-to-build"
                  },
                  "sort" : [
                    "1"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658668",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "a-step-by-step-guide-to-designing-your-home"
                  },
                  "sort" : [
                    "2"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658670",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "building-an-off-the-plan-home-guide"
                  },
                  "sort" : [
                    "3"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658672",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "a-guide-to-building-a-custom-home"
                  },
                  "sort" : [
                    "4"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658674",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "choosing-a-builder-guide"
                  },
                  "sort" : [
                    "5"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658676",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "the-costs-of-building-a-home"
                  },
                  "sort" : [
                    "6"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658678",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "finalising-your-build"
                  },
                  "sort" : [
                    "7"
                  ]
                }
              ]
            }
          }
        },
        {
          "key" : "buying-apartments-guide",
          "doc_count" : 6,
          "agg_source" : {
            "hits" : {
              "total" : {
                "value" : 6,
                "relation" : "eq"
              },
              "max_score" : null,
              "hits" : [
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658681",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "everything-you-need-to-know-about-buying-an-apartment"
                  },
                  "sort" : [
                    "1"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658686",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "buying-an-off-the-plan-apartment"
                  },
                  "sort" : [
                    "2"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658691",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "buying-established-apartment-guide"
                  },
                  "sort" : [
                    "3"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658693",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "settlement-for-your-apartment"
                  },
                  "sort" : [
                    "4"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658793",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "financing-your-apartment-guide"
                  },
                  "sort" : [
                    "5"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "658795",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "moving-into-your-new-apartment-guide"
                  },
                  "sort" : [
                    "6"
                  ]
                }
              ]
            }
          }
        },
        {
          "key" : "selling-guide",
          "doc_count" : 4,
          "agg_source" : {
            "hits" : {
              "total" : {
                "value" : 4,
                "relation" : "eq"
              },
              "max_score" : null,
              "hits" : [
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "630638",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "i-want-to-sell-my-house-where-do-i-start"
                  },
                  "sort" : [
                    "1"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "630647",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "choosing-real-estate-agent-to-sell-your-house-guide"
                  },
                  "sort" : [
                    "2"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "630651",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "preparing-your-house-for-sale-guide"
                  },
                  "sort" : [
                    "3"
                  ]
                },
                {
                  "_index" : "news-lifestyle-post-v2",
                  "_type" : "_doc",
                  "_id" : "630739",
                  "_score" : null,
                  "_source" : {
                    "post_name" : "selling-house-and-next-steps-guide"
                  },
                  "sort" : [
                    "4"
                  ]
                }
              ]
            }
          }
        }
      ]
    }
  }
}
```：
