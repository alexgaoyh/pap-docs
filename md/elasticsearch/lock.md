# ES 锁
    悲观锁：
        全局锁：    可以使用一个专门的新索引名称，用来进行加锁操作，获取到锁之后才可以进行数据操作，执行完毕后删除锁。
            加锁：
                curl --location --request POST 'http://localhost:9200/student_lock/_doc/global/_create' \
                    --header 'Content-Type: application/json' \
                    --data-raw '{
                    
                    }'
            移除锁：
                curl --location --request DELETE 'http://localhost:9200/student_lock/_doc/global'

        document锁：  
            加锁：
                curl --location --request POST 'http://localhost:9200/student_lock/_doc/1/_update' \
                    --header 'Content-Type: application/json' \
                    --data-raw '{
                        "upsert": {
                            "thread_id": "QWER-1234-ASDF-5678-2"
                        },
                        "script": {
                            "lang": "painless",
                            "source": "if(ctx._source.thread_id != params.thread_id){ Debug.explain(ctx._source.thread_id) } else { ctx.op = '\''noop'\'' } ",
                            "params": {
                                "thread_id": "QWER-1234-ASDF-5678-2"
                            }
                        }
                    }'
            查看锁：
                curl --location --request GET 'http://localhost:9200/student_lock/_doc/1'
            移除锁：
                curl --location --request DELETE 'http://localhost:9200/student_lock/_doc/1'
        
        读写锁(共享锁/排它锁)：
            共享锁(读锁)加锁:
                curl --location --request POST 'http://localhost:9200/student_lock/_doc/global/_update' \
                    --header 'Content-Type: application/json' \
                    --data-raw '{
                        "upsert": {
                            "lock_type": "shared",
                            "lock_count": 1
                        },
                        "script": {
                            "lang": "painless",
                            "source": "if ( ctx._source.lock_type == '\''exclusive'\'' ) { Debug.explain(ctx._source.lock_type) } else { ctx._source.lock_count++ } "
                        }
                    }'
            共享锁(读锁)释放:
                curl --location --request POST 'http://localhost:9200/student_lock/_doc/global/_update' \
                    --header 'Content-Type: application/json' \
                    --data-raw '{
                        "script": {
                            "lang": "painless",
                            "source": "if(--ctx._source.lock_count == 0){ ctx.op = '\''delete'\''} "
                        }
                    }'
            排他锁(写锁)加锁:
                curl --location --request POST 'http://localhost:9200/student_lock/_doc/global/_create' \
                    --header 'Content-Type: application/json' \
                    --data-raw '{
                        "lock_type": "exclusive"
                    }'
            排他锁(写锁)释放:
                curl --location --request DELETE 'http://localhost:9200/student_lock/_doc/global'
