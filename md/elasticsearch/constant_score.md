
# constant_score 使用

    搜索对原始的评分不感兴趣，只对关键词是否出现感兴趣，只要出现就行，并不关注TF/IDF 。

    {
        "query": {
            "bool": {
                "should": [
                    {
                        "constant_score": {
                            "boost": 1000,
                            "filter": {
                                "match": {
                                    "studentTags": "A"
                                }
                            }
                        }
                    },
                    {
                        "constant_score": {
                            "boost": 100,
                            "filter": {
                                "match": {
                                    "studentTags": "B"
                                }
                            }
                        }
                    },
                    {
                        "constant_score": {
                            "boost": 10,
                            "filter": {
                                "match": {
                                    "studentTags": "C"
                                }
                            }
                        }
                    },
                    {
                        "constant_score": {
                            "boost": 1,
                            "filter": {
                                "match": {
                                    "studentTags": "D"
                                }
                            }
                        }
                    }
                ]
            }
        }
    }
