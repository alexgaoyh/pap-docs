# ElasticSearch 自定义相似度插件

## 自定义相似度算法(只考虑词频)

&ensp;&ensp;在使用Elasticsearch的时候，针对排序结果，有些时候只关注对应的词出现的次数，相当于只考虑词频，这个时候就可以使用当前的插件。
&ensp;&ensp;当前插件继承了 TFIDFSimilarity 类， TfSimilarity 只考虑了词频，并将其注册到插件中。
&ensp;&ensp;实现结果如下，前两个代码段落分别是 mapping setting 配置文件，第三个代码段是请求，第四个代码段是结果。
&ensp;&ensp;详细查看第四个代码段落的 _score 得分，发现 _score 的值等于请求参数'效果'在文本中出现的次数，至此证明当前插件有效。

```html
"studentIntro": {
    "type": "text",
    "analyzer": "ik_max_word",
    "search_analyzer": "ik_smart",
    "similarity": "tf_similarity"
},
```
```html
{
    "similarity": {
        "tf_similarity": {
            "type": "tf_similarity"
        }
    }
}
```
```html
{
    "size": 100,
    "query": {
        "bool": {
            "should": [
                {
                    "match": {
                        "studentIntro": {
                            "query": "效果"
                        }
                    }
                }
            ]
        }
    }
}
```
```html
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 6,
            "relation": "eq"
        },
        "max_score": 2.0,
        "hits": [
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "26",
                "_score": 2.0,
                "_source": {
                    "_class": "com.pap.es.domain.Student",
                    "id": "26",
                    "studentName": "洋柿子",
                    "studentNo": "GAO202209",
                    "studentAge": 11,
                    "studentBirth": "2023-04-13 03:45:02",
                    "studentIntro": "我是效果，我要看看分词的效果",
                    "studentTags": "D",
                    "gradePoint": 2.35
                }
            },
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "4",
                "_score": 1.0,
                "_source": {
                    "_class": "com.pap.es.domain.Student",
                    "id": "4",
                    "studentName": "洋柿子",
                    "studentNo": "GAO202209",
                    "studentAge": 11,
                    "studentBirth": "2023-04-13 03:44:58",
                    "studentIntro": "我是洋柿子，我要看看分词的效果",
                    "studentTags": "D",
                    "gradePoint": 2.35
                }
            },
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "3",
                "_score": 1.0,
                "_source": {
                    "_class": "com.pap.es.domain.Student",
                    "id": "3",
                    "studentName": "圣女果",
                    "studentNo": "123GAO56",
                    "studentAge": 13,
                    "studentBirth": "2023-04-13 03:44:58",
                    "studentIntro": "我是圣女果，我要看看分词的效果",
                    "studentTags": "C",
                    "gradePoint": 2.25
                }
            },
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "_class": "com.pap.es.domain.Student",
                    "id": "1",
                    "studentName": "西红柿",
                    "studentNo": "YI123GAO",
                    "studentAge": 12,
                    "studentBirth": "2023-04-13 03:44:57",
                    "studentIntro": "我是西红柿，我要看看分词的效果",
                    "studentTags": "A",
                    "gradePoint": 2.23
                }
            },
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "2",
                "_score": 1.0,
                "_source": {
                    "_class": "com.pap.es.domain.Student",
                    "id": "2",
                    "studentName": "番茄",
                    "studentNo": "GAO456YH",
                    "studentAge": 12,
                    "studentBirth": "2023-04-13 03:44:58",
                    "studentIntro": "我是番茄，我要看看分词的效果",
                    "studentTags": "B",
                    "gradePoint": 2.33
                }
            },
            {
                "_index": "student",
                "_type": "_doc",
                "_id": "6",
                "_score": 1.0,
                "_source": {
                    "_class": "com.pap.es.domain.Student",
                    "id": "6",
                    "studentName": "效果",
                    "studentNo": "20191202",
                    "studentAge": 14,
                    "studentBirth": "2023-04-13 03:44:59",
                    "studentIntro": "效果",
                    "studentTags": "A,B,C",
                    "gradePoint": 2.13
                }
            }
        ]
    }
}
```

## 使用方法

1. clone 当前代码，修改 pom.xml 文件中对应的 elasticsearch 版本；
2. mvn clean package 打包；
3. 防止到 elasticsearch 目录的 plugins 文件夹下，并重启 ES；

## 相关链接
1. https://gitee.com/alexgaoyh/elasticsearch-similarity-tf
2. https://gitee.com/alexgaoyh/elasticsearch-similarity-tf/releases/tag/7.17.6
