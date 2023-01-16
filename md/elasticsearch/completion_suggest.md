# ES completion suggest

    在 studentIntro.suggest 这里字段中，搜索以 '我是' 开头的数据    

    curl --location --request POST 'http://localhost:9200/student/_search' \
        --header 'Content-Type: application/json' \
        --data-raw '{
            "suggest": {
                "intro-suggest": {
                    "prefix": "我是",
                    "completion": {
                    "field": "studentIntro.suggest"
                    }
                }
            }
        }'
