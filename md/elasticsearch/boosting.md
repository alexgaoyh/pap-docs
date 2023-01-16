# 相关度分数优化 negative boost(不包含)  boosting.positive boosting.negative

    搜索 studentIntro 包含 '智能',尽量不包含 '手机' 的 doc,如果包含了 '手机'，不会说排除掉这个 doc,而是说将这个 doc 的分数降低.
    negative_boost ： 否定的权重,包含了 negative term 的 doc,分数乘以 negative boost,分数降低.
    {
        "query": {
            "boosting": {
                "positive": {
                    "match": {
                        "studentIntro": "智能"
                    }
                },
                "negative": {
                    "match": {
                        "studentIntro": "手机"
                    }
                },
                "negative_boost": 0.2
            }
        }
    }
