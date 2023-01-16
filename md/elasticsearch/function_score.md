# function_score 自定义打分
    ES 会为 query 的每个文档计算一个相关度得分 score ，并默认按照 score 从高到低的顺序返回搜索结果，在很多场景下，我们不仅需要搜索到匹配的结果，还需要能够按照某种方式对搜索结果重新打分排序。
    Eg： 
        搜索具有某个关键词的文档，同时考虑到文档的时效性进行综合排序；
        搜索某个旅游景点附近的酒店，同时根据距离远近和价格等因素综合排序；
        搜索标题包含 elasticsearch 的文章，同时根据浏览次数和点赞数进行综合排序；

    
        如下的搜索，对搜索结果重新打分，根据 studentBirth.date 进行 高斯函数 衰减， 中心点是 origin 日期，scale 意味着日期范围是以orgin为中心点的scale范围内的文档分数权重是 1 ，日期在 scale + offset = 15d 之外的文档权重是 0.5 。
        {
            "query": {
                "function_score": {
                    "query": {
                        "match": {
                            "studentName": "近似"
                        }
                    },
                    "gauss": {
                        "studentBirth.date": {
                            "origin": "2022-11-30",
                            "scale": "10d",
                            "offset": "5d",
                            "decay": 0.5
                        }
                    }
                }
            }
        }

        如下的搜索，可以实现类似 随机抽样 的功能， 其中 seed 可以使用当前用户或者设备的标识符，尽量保证每次获取数据的结果是一致的。
        {
            "query": {
                "function_score": {
                    "random_score": {
                        "seed": "${userId}",
                        "field": "_seq_no"
                    }
                }
            }
        }

        如下的搜索，使用 functions 定义多个不同的权重规则
        {
            "query": {
                "function_score": {
                    "query": {
                        "match": {
                            "studentName": "近似"
                        }
                    },
                    "functions": [
                        {
                            "filter": {
                                "term": {
                                    "studentAge": 14
                                }
                            },
                            "weight": 2
                        },
                        {
                            "filter": {
                                "match": {
                                    "studentIntro": "大屏"
                                }
                            },
                            "weight": 1.5
                        }
                    ]
                }
            }
        }
