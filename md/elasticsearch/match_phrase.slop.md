# match_phrase 的 slop
    1、尝试着把 match_phrase.slop 的数值调整大一点，你会发现靠得越近的（slop 相对小的）得分会越高，更类似于 近似匹配(proximity match)
    
        {
            "query": {
                "match_phrase": {
                    "studentIntro": {
                        "query": "智能 手机",
                        "slop": 3
                    }
                }
            }
        }

    2、在进行搜索的过程中，牵扯到搜索结果召回率与精准度的平衡，如上命令，如果doc中只有‘智能’或者‘手机’，而不是都包含，其实也可以搜索出来，只是建议得分较低，
        基于此，可以混合使用 must 和 match_phrase， 这样的话，就对召回率和精准度进行了平衡。
        
        {
            "query": {
                "bool": {
                    "must": [
                        {
                            "match": {
                                "studentIntro": "智能 手机"
                            }
                        }
                    ],
                    "should": [
                        {
                            "match_phrase": {
                                "studentIntro": {
                                    "query": "智能 手机",
                                    "slop": 3
                                }
                            }
                        }
                    ]
                }
            }
        }

    3、rescore 重打分
        如上2描述的方法，可以对其进行优化，提升执行效率，比如使用 rescoring：
        优化 proximity match 的性能，一般就是减少要进行 proximity match 搜索的 document 数量。 
        主要思路就是，用 match query 先过滤出需要的数据，然后再用 proximity match 来根据 term 距离提高 doc 的分数， 
        同时 proximity match 只针对每个 shard 的分数排名前 n 个 doc 起作用，来重新调整它们的分数， 这个过程称之为 rescoring，重计分。
        因为一般用户会分页查询，只会看到前几页的数据，所以不需要对所有结果进行 proximity match 操作。

        {
            "query": {
                "match": {
                    "studentIntro": "智能 手机"
                }
            },
            "rescore": {
                "window_size": 100,
                "query": {
                    "rescore_query": {
                        "match_phrase": {
                            "studentIntro": {
                                "query": "智能 手机",
                                "slop": 3
                            }
                        }
                    }
                }
            }
        }
