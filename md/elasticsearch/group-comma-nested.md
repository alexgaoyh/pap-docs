# ElasticSearch 分组统计

## 逗号分割的字符串，如何进行分组统计

&ensp;&ensp;在使用Elasticsearch的时候，经常会遇到类似标签的需求，比如给学生信息打标签，并且使用逗号分割的字符串进行存储，后期如果遇到需要根据标签统计学生数量的需求，则可以使用如下的命令进行处理。
&ensp;&ensp;前两个代码段落分别是 mapping、setting的配置，第三个代码段是请求命令，第四个代码段是分组结果。

```html
 "studentTags": {
   "type": "text",
   "analyzer": "comma",
   "search_analyzer": "comma"
 }
```
```html
{
  "analysis": {
    "filter": {
    },
    "analyzer": {
      "comma": {
        "type": "pattern",
        "pattern":","
      }
    },
    "char_filter": {
    },
    "tokenizer": {
    }
  }
}
```
```html
{
    "size": 0,
    "aggs": {
        "group_by_field": {
            "terms": {
                "script": {
                    "source": "if (params['_source']['studentTags'] != null) { params['_source']['studentTags'].splitOnToken(',') }",
                    "lang": "painless"
                },
                "size": 10
            }
        }
    }
}
```
```html
{
    "took": 9,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 27,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "group_by_field": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": "A",
                    "doc_count": 19
                },
                {
                    "key": "B",
                    "doc_count": 18
                },
                {
                    "key": "C",
                    "doc_count": 13
                },
                {
                    "key": "D",
                    "doc_count": 12
                }
            ]
        }
    }
}
```

## Nested对象，如何进行分组统计

&ensp;&ensp;在使用Elasticsearch的时候，如果遇到nested对象，并且想对nested对象进行分组统计的话，可以按照如下方式进行处理。
&ensp;&ensp;第一个代码段落分别是 mapping，第二个代码段是请求命令，第三个代码段是分组结果。

```html
"mathScoreNestedList": {
    "type": "nested",
    "properties": {
            "score": {
            "type": "integer"
        },
            "halfYear": {
            "type": "keyword"
        }
    }
},
```
```html
{
    "size": 0,
    "aggs": {
        "labels_nested": {
            "nested": {
                "path": "mathScoreNestedList"
            },
            "aggs": {
                "nested_score": {
                    "terms": {
                        "field": "mathScoreNestedList.halfYear"
                    }
                }
            }
        }
    }
}
```
```html
{
    "took": 32,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 27,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "labels_nested": {
            "doc_count": 7,
            "nested_score": {
                "doc_count_error_upper_bound": 0,
                "sum_other_doc_count": 0,
                "buckets": [
                    {
                        "key": "202201",
                        "doc_count": 5
                    },
                    {
                        "key": "202207",
                        "doc_count": 2
                    }
                ]
            }
        }
    }
}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
