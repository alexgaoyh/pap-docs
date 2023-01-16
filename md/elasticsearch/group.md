# 分组
    ES为了满足搜索的实时性，在聚合分析的一些场景会通过损失精准度的方式加快结果的返回。这其实ES在实时性和精准度中间的权衡。
    需要明确的是，并不是所有的聚合分析都会损失精准度，比如min,max等这些就没有精准度的问题。

    DSL 根据 studentAge studentBirth 进行分组 
        {
            "size": 0,
            "aggs": {
                "studentAge_Name_aggs_terms": {
                    "terms": {
                        "field": "studentAge",
                        "order": {
                            "_key": "desc"
                        }
                    },
                    "aggs": {
                        "studentName_aggs_terms": {
                            "terms": {
                                "field": "studentBirth"
                            }
                        }
                    }
                }
            }
        }

    统计每个年龄段下的平均出生日期，对每个 bucket(studentAge_aggs_terms) 的数据进行平均值计算
    其中 order 部分可以改为  "studentBirth_avg_terms": "asc"  意味着将按照 平均出生时间 进行递增排序  对于每个聚合操作的结果都可以进行定制排序
        {
            "size": 0,
            "aggs": {
                "studentAge_aggs_terms": {
                    "terms": {
                        "field": "studentAge",
                        "order": {
                            "_key": "desc"
                        }
                    },
                    "aggs": {
                        "studentBirth_avg_terms": {
                            "avg": {
                                "field": "studentBirth.date"
                            }
                        }
                    }
                }
            }
        }

    区间分组语法： histogram date_histogram
        {
            "size": 0,
            "aggs": {
                "sales": {
                    "date_histogram": {
                        "field": "studentBirth.date",
                        "interval": "month",
                        "format": "yyyy-MM-dd",
                        "extended_bounds": {
                            "min": "2022-10-01",
                            "max": "2023-01-01"
                        }
                    }
                }
            }
        }

    每个出生月份下，不同的年龄数量统计(按照出生日期的月份进行划分，获取下面不同年龄的数量)(cardinality)
        {
            "size": 0,
            "aggs": {
                "group_by_date": {
                    "date_histogram": {
                        "field": "studentBirth.date",
                        "interval": "month"
                    },
                    "aggs": {
                        "distinct_studentAge": {
                            "cardinality": {
                                "field": "studentAge"
                            }
                        }
                    }
                }
            }
        }

    出生日期在过去 50 天、150 天的平均年龄 : bucket.filter 可以让你在一个聚合分析请求中完成不同数据的过滤统计, 对不同 bucket 下的 aggs 进行 filter
        {
            "size": 0,
            "aggs": {
                "recent_50d": {
                    "filter": {
                        "range": {
                            "studentBirth.date": {
                                "gte": "now-50d"
                            }
                        }
                    },
                    "aggs": {
                        "recent_50d_avg_price": {
                            "avg": {
                                "field": "studentAge"
                            }
                        }
                    }
                },
                "recent_150d": {
                    "filter": {
                        "range": {
                            "studentBirth.date": {
                                "gte": "now-150d"
                            }
                        }
                    },
                    "aggs": {
                        "recent_150d_avg_price": {
                            "avg": {
                                "field": "studentAge"
                            }
                        }
                    }
                }
            }
        }

    percentiles 百分比算法 : topN 前多少的数据的平均年龄 : 前50%的学生的平均年龄，前95%的学生的平均年龄，前99%的学生的平均年龄
        {
            "size": 0,
            "aggs": {
                "studentAge_percentiles": {
                    "percentiles": {
                        "field": "studentAge",
                        "percents": [
                            50,
                            95,
                            99
                        ]
                    }
                },
                "studentAge_avg": {
                    "avg": {
                        "field": "studentAge"
                    }
                }
            }
        }

    percentile_ranks : 按照 studentAge 进行分组，并统计在不同数据范围的占比， 比如在10岁以下的占比多少，在20岁以下的占比多少
        {
            "size": 0,
            "aggs": {
                "group_by_studentAge": {
                    "terms": {
                        "field": "studentAge"
                    },
                    "aggs": {
                        "studentAge_percentile_ranks": {
                            "percentile_ranks": {
                                "field": "studentAge",
                                "values": [
                                    10,
                                    20
                                ]
                            }
                        }
                    }
                }
            }
        }


    分组查询 : bucket 优化机制 : 从深度优先到广度优先
        collect_mode : 默认是深度优先的方式去执行聚合操作的，可以修改为广度优先

    aggs : top_hits : 按照年龄进行分组，并取出来分组下指定个数的学生姓名信息
        {
            "size": 0,
            "aggs": {
                "studentAge_aggs_terms": {
                    "terms": {
                        "field": "studentAge",
                        "order": {
                            "_key": "asc"
                        }
                    },
                    "aggs": {
                        "studentName_top": {
                            "top_hits": {
                                "_source": {
                                    "include": [
                                        "studentName"
                                    ]
                                },
                                "size": 2
                            }
                        }
                    }
                }
            }
        }
