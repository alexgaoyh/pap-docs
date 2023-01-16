# Nested Object

    nested类型允许对象数组以相互独立的方式进行索引
    
        {
            "query": {
                "bool": {
                    "must": [
                        {
                            "match": {
                                "mathScoreList.halfYear": "202201"
                            }
                        },
                        {
                            "match": {
                                "mathScoreList.score": 85
                            }
                        }
                    ]
                }
            }
        }
    
        {
            "query": {
                "nested": {
                    "path": "mathScoreNestedList",
                    "query": {
                        "bool": {
                            "must": [
                                {
                                    "match": {
                                        "mathScoreNestedList.halfYear": "202201"
                                    }
                                },
                                {
                                    "match": {
                                        "mathScoreNestedList.score": 85
                                    }
                                }
                            ]
                        }
                    }
                }
            }
        }

    Nested Object 搜索：

        按照成绩的学年进行分组，并计算平均成绩：
            {
                "size": 0,
                "aggs": {
                    "score_nested": {
                        "nested": {
                            "path": "mathScoreNestedList"
                        },
                        "aggs": {
                            "group_by_halfYear": {
                                "terms": {
                                    "field": "mathScoreNestedList.halfYear"
                                },
                                "aggs": {
                                    "avg_score": {
                                        "avg": {
                                            "field": "mathScoreNestedList.score"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        
        根据学生的成绩，按10分一个范围进行查询，并获取每个分数段分组下的学生编号 ("reverse_nested": {}, 在nested中的时候需要引用外部的字段需要额外的配置 reverse_nested)
            {
                "size": 0,
                "aggs": {
                    "mathScoreNestedList_nested": {
                        "nested": {
                            "path": "mathScoreNestedList"
                        },
                        "aggs": {
                            "group_by_score": {
                                "histogram": {
                                    "field": "mathScoreNestedList.score",
                                    "interval": 10
                                },
                                "aggs": {
                                    "reverse_path": {
                                        "reverse_nested": {},
                                        "aggs": {
                                            "group_by_studentNo": {
                                                "terms": {
                                                    "field": "studentNo.keyword"
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
