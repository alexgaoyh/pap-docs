
# 订单号/合同号的模糊查询
    举一个场景，用户只知道订单号/合同号的一部分数据，并不知道所有的数据，之后开始搜索，期望把包含这些字符的数据都返回。
    当前采用 ngram 对原始编号进行分词, 并设定 min_gram == max_gram == 1, 这样每一个字母或者数字都会被拆分，
    查询的时候使用 query.match , 并设置 query.match.operator = and , 这样每一个用户的输入都会进行匹配。

    {
        "size": 10,
        "query": {
            "match": {
                "studentNo": {
                    "query": "123",
                    "operator" : "and"
                }
            }
        }
    }

    还可以采用 regexp 的方式，举例如果用户的输入使用空格分开，那么可以在前后和空格处改为 .*  使用 regexp 的方式进行搜索，这样的话还匹配对应的顺序.

    {
        "size": 10,
        "query": {
            "regexp": {
                "studentNo.keyword": ".*123.*5.*"
            }
        }
    }


    在搜索的过程中，可能会存在对搜索结果排序的情况，比如按照字符出现的顺序从左到右进行排序，如果排序相同的话，则按照字典值进行排序，详见如下命令：

    {
        "query": {
            "regexp": {
                "studentNo.keyword": ".*123.*"
            }
        },
        "sort": [
            {
                "_script": {
                    "type": "number",
                    "script": {
                        "lang": "painless",
                        "source": "doc['studentNo.keyword'].value.indexOf(params.studentNo)",
                        "params": {
                            "studentNo": "123"
                        }
                    },
                    "order": "asc"
                }
            },
            {
                "studentNo.keyword": "asc"
            }
        ],
        "track_scores": true
    }

    自定义排序与评分： 在如上的场景下，如果按照搜索的关键词的前后顺序进行排序，存在多个搜索关键词的话，可以采用如下的方式，获取到每一个关键词，之后判断每一组关键字出现的顺序并相加，最为最终的得分。

    {
        "query": {
            "regexp": {
                "studentNo.keyword": ".*ALEX.*022.*20.*"
            }
        },
        "sort": [
            {
                "_script": {
                    "type": "number",
                    "script": {
                        "lang": "painless",
                        "source": "int total = 0; int leng = params.studentNo.splitOnToken('.*').length; for(int idx =  1; idx < leng - 1; idx++) { total += doc['studentNo.keyword'].value.indexOf(params.studentNo.splitOnToken('.*')[idx]) } return total;",
                        "params": {
                            "studentNo": ".*ALEX.*022.*20.*"
                        }
                    },
                    "order": "asc"
                }
            },
            {
                "studentNo.keyword": "asc"
            }
        ],
        "track_scores": true
    }

# ES 重新添加字段，查询类型与字段赋值

    1、索引创建之后，如果想要再增加字段的话，是有一定要求的，此处不过多进行描述，针对原始的 student-mapping.json 文件，如果针对 studentTags 字段，需要增加 type = keyword 的节点（fields），如下所示

        "studentTags": {
          "type": "text",
          "analyzer": "comma",
          "search_analyzer": "comma",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        }
    
    2、原始的 spring boot elasticsearch @Document 的方式，是在项目启动的时候创建索引，但是如果索引发生变化，并不能重新按照新的格式刷新索引，可以按照如下方式刷新一下索引的 mapping。
    测试方法可以在更改索引mapping文件的前后，在 elasticsearch-head 插件中，找到对应的索引，点击 信息->索引信息， 查看执行前后的索引信息。

        InputStream inputStream = new ClassPathResource("doc/student-mapping.json").getInputStream();

        PutMappingRequest putMappingRequest = PutMappingRequest.of(
                m -> m.
                        index("student").
                        withJson(inputStream));

        PutMappingResponse putMappingResponse = elasticsearchClient.indices().putMapping(putMappingRequest);

    3、更新过索引之后，可以测试一下新生成的索引字段是否有值，可以执行如下DSL命令：

        {
            "query": {
                "bool": {
                    "must_not": {
                        "exists": {
                            "field": "studentTags.keyword"
                        }
                    }
                }
            }
        }

    4、字段赋值，可以直接执行 POST {HOST:PORT}/student/_update_by_query?refresh , 其中传参为 {} 空，之后再次执行第三条的命令，就发现 studentTags.keyword 就已经被赋值成功

        curl --location --request POST 'http://localhost:9200/student/_update_by_query?refresh' \
            --header 'Content-Type: application/json' \
            --data-raw '{
            }'


# ES 字符串/数组 长度和固定值判断。

    1、如果遇到数组类型，想要判断数组的长度，可以使用 script ， 应用场景类似判断数组下是否只有指定数量的数据，第一个命令是数组类型，第二个命令是逗号分割的字符串
    
    {
        "query": {
            "script": {
                "script": "doc['mathScoreList.score'].length === 1" 
            }
        }
    }

    {
        "query": {
            "script": {
                "script": "doc['studentTags.keyword'].value.splitOnToken(',').length === 1" 
            }
        }
    }

    2、字符串判断是否为指定的值

    {
        "query": {
            "script": {
                "script": "doc['studentTags.keyword'].value.equals('D')" 
            }
        }
    }


# ES group by having order by limit
    SELECT studentAge,COUNT(DISTINCT studentBirth) studentBirth_count FROM student GROUP BY studentAge HAVING studentBirth_count > 1 ORDER BY studentBirth_count desc LIMIT 3

    {
        "size": 0,
        "aggs": {
            "models": {
                "terms": {
                    "field": "studentAge",
                    "order": {
                        "_key": "asc"
                    }
                },
                "aggs": {
                    "studentBirth_count": {
                        "cardinality": {
                            "field": "studentBirth"
                        }
                    },
                    "studentBirth_count_count_filter": {
                        "bucket_selector": {
                            "buckets_path": {
                                "studentBirthCount": "studentBirth_count"
                            },
                            "script": "params.studentBirthCount>1"
                        }
                    },
                    "studentBirth_count_count_sort": {
                        "bucket_sort": {
                            "sort": {
                                "studentBirth_count": "desc"
                            },
                            "size": 3
                        }
                    }
                }
            }
        }
    }
