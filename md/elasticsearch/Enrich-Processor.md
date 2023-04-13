# Elasticsearch 利用 Enrich Processor 实现 MYSQL Join 的操作，支持Nested类型，具体应用。

    在关系型数据库中，表关联是非常常见的操作，放到 Elasticsearch 的场景，相当于跨索引进行数据关联，本文使用 Enrich Processor，在数据预处理阶段把数据进行维护，最终使得数据在存储的时候，就是已经进行过数据关联的。
    
    举一个很常见的 省市区行政区域扩充的例子(同理类似于 字典 处理的场景)，在保存数据的时候，把关联的数据查询出来并存到 document 中。

    1、初始化：
        假设初始化一个索引： dict-detail，索引中包含的数据值如下所示，都是 keyword 类型，数据值如下所示。
    
            public DictDetail(String dictDetailId, String dictId, String dictCode, String dictName, String dictDetailCode, String dictDetailName) {
                this.dictDetailId = dictDetailId;
                this.dictId = dictId;
                this.dictCode = dictCode;
                this.dictName = dictName;
                this.dictDetailCode = dictDetailCode;
                this.dictDetailName = dictDetailName;
            }
    
            DictDetail dictDetail2 = new DictDetail("410102", "410100", "410100", "郑州市", "410102", "中原区");
            DictDetail dictDetail3 = new DictDetail("410103", "410100", "410100", "郑州市", "410103", "二七区");
            DictDetail dictDetail4 = new DictDetail("410104", "410100", "410100", "郑州市", "410104", "管城回族区");
            DictDetail dictDetail5 = new DictDetail("410105", "410100", "410100", "郑州市", "410105", "金水区");
            DictDetail dictDetail6 = new DictDetail("410106", "410100", "410100", "郑州市", "410106", "上街区");
            DictDetail dictDetail8 = new DictDetail("410108", "410100", "410100", "郑州市", "410108", "惠济区");

    2、执行效果展示（需要优先执行 3.1、3.2、3.3 部分的命令）：
        2.1、文档创建：
            PUT http://localhost:9200/test-dict-detail/_doc/1?pipeline=enrich-dict-detail-id-policy-pipeline
                {
                    "dictDetailId": "410102",
                    "comment": "希望创建之后能够出现dictDetailId对应的扩展字典信息dictDetailId_enriched",
                    "address": [
                        {
                            "dictDetailId": "410102"
                        },
                        {
                            "dictDetailId": "410103"
                        }
                    ]
                }

        2.2、文档查看： 注意保存到索引中的文档增加了两个属性，一个是 dictDetailId 扩展出来的 dictDetailId_enriched，  另一个是 address.dictDetailId 扩展出来的 address._dictDetailIdObj

            GET http://localhost:9200/test-dict-detail/_doc/1
                
                {
                    "_index": "test-dict-detail",
                    "_type": "_doc",
                    "_id": "1",
                    "_version": 21,
                    "_seq_no": 20,
                    "_primary_term": 2,
                    "found": true,
                    "_source": {
                        "address": [
                            {
                                "_dictDetailIdObj": {
                                    "dictDetailId": "410102",
                                    "dictName": "郑州市",
                                    "dictId": "410100",
                                    "dictDetailName": "中原区",
                                    "dictCode": "410100",
                                    "dictDetailCode": "410102"
                                },
                                "dictDetailId": "410102"
                            },
                            {
                                "_dictDetailIdObj": {
                                    "dictDetailId": "410103",
                                    "dictName": "郑州市",
                                    "dictId": "410100",
                                    "dictDetailName": "二七区",
                                    "dictCode": "410100",
                                    "dictDetailCode": "410103"
                                },
                                "dictDetailId": "410103"
                            }
                        ],
                        "dictDetailId": "410102",
                        "comment": "希望创建之后能够出现dictDetailId对应的扩展字典信息dictDetailId_enriched",
                        "dictDetailId_enriched": {
                            "dictDetailId": "410102",
                            "dictName": "郑州市",
                            "dictId": "410100",
                            "dictDetailName": "中原区",
                            "dictCode": "410100",
                            "dictDetailCode": "410102"
                        }
                    }
                }


    3、实现方法：
        针对 enrich processor 的概念，很多文档都进行了描述，本文结合代码进行详细说明：
        
        3.1、创建策略，该策略定义我们将使用哪个字段将主数据与输入数据流中的文档进行匹配。

            match ： policy 类型，除了传统的match类型，还有应用于地理位置场景的：geo_match。
            indices : 一个或多个源索引的列表，存储的是待 enrich 扩展的数据。
            match_field ： 源索引中用于匹配传入文档的匹配字段。
            enrich_field ： 源索引中的字段列表，用于添加到新传入的文档中。

            PUT http://localhost:9200/_enrich/policy/enrich-dict-detail-id-policy
                {
                    "match": {
                        "indices": "dict-detail",
                        "match_field": "dictDetailId",
                        "enrich_fields": [
                            "dictId",
                            "dictCode",
                            "dictName",
                            "dictDetailCode",
                            "dictDetailName"
                        ]
                    }
                }
        
        3.2、运行策略
            PUT http://localhost:9200/_enrich/policy/enrich-dict-detail-id-policy/_execute

        3.3、创建一个 pipeline
            注意这里针对普通field属性和Nested-Object属性都进行了描述。
            
            3.3.1、processors 中的第一个 enrich 就是针对 普通field属性，只要匹配到 dictDetailId 属性，就对其扩充生成 dictDetailId_enriched， 详见如上 2.2 部分的结果；
            3.3.2、processors 中的剩余的操作，是针对 Nested-Object进行的操作，只要匹配到 address.dictDetailId，就对其扩充生成 address._dictDetailIdObj，详见如上 2.2 部分的结果；
                3.3.2.1、首先使用 foreach 遍历 address 属性，取出来其中的 dictDetailId属性，生成 addressDictDetailIds 数组；
                3.3.2.2、使用第一步生成的 addressDictDetailIds 数组，进行扩充生成 addressDictDetailIds_enriched 数组对象，这里的数组对象就是已经进行属性扩充的对象；
                3.3.2.3、删除 addressDictDetailIds 数组；
                3.3.2.4、将 addressDictDetailIds_enriched 数组对象 与 原始的 address 数组对象进行合并，在 address 数组对象中增加 _dictDetailIdObj 属性；
                3.3.2.5、删除 addressDictDetailIds_enriched 数组对象；

            PUT http://localhost:9200/_ingest/pipeline/enrich-dict-detail-id-policy-pipeline

            {
                "description": "Enrich dict-detail-id information",
                "processors": [
                    {
                        "enrich": {
                            "description": "单独对象：针对 field 属性进行扩充,生成 target_field 对象",
                            "policy_name": "enrich-dict-detail-id-policy",
                            "field": "dictDetailId",
                            "target_field": "dictDetailId_enriched",
                            "max_matches": "1",
                            "ignore_failure": true
                        }
                    },
                    {
                        "foreach": {
                            "field": "address",
                            "ignore_failure": true,
                            "description": "Nested-Object-Enrich:1、遍历 field 对象, 生成 field 数组",
                            "processor": {
                                "append": {
                                    "field": "addressDictDetailIds",
                                    "value": [
                                        "{{_ingest._value.dictDetailId}}"
                                    ]
                                }
                            }
                        }
                    },
                    {
                        "enrich": {
                            "description": "Nested-Object-Enrich:2、针对第一步生成的数组，进行扩充生成 target_field 数组对象",
                            "policy_name": "enrich-dict-detail-id-policy",
                            "field": "addressDictDetailIds",
                            "target_field": "addressDictDetailIds_enriched",
                            "max_matches": "128",
                            "ignore_failure": true
                        }
                    },
                    {
                        "remove": {
                            "description": "Nested-Object-Enrich:3、针对第一步生成的数组，已经经过了第二步的处理，故删除第一步临时生成的数组对象",
                            "ignore_failure": true,
                            "field": [
                                "addressDictDetailIds"
                            ]
                        }
                    },
                    {
                        "script": {
                            "description": "Nested-Object-Enrich:4、将原始对象和第3步生成的数组对象进行合并",
                            "lang": "painless",
                            "ignore_failure": true,
                            "source": "for (int i = 0; i < ctx.address.length; i++) { ctx.address[i]._dictDetailIdObj = ctx.addressDictDetailIds_enriched[i] }"
                        }
                    },
                    {
                        "remove": {
                            "description": "Nested-Object-Enrich:5、删除第二步扩充生成的对象",
                            "field": [
                                "addressDictDetailIds_enriched"
                            ],
                            "ignore_failure": true
                        }
                    }
                ]
            }
    
        3.4、就可以使用 2.1 和 2.2 两个部分的命令，创建文档和查看文档，至此类似 Mysql 的 join 操作就已经实现；

    4、其他命令：
        删除 pipeline
            DELETE http://localhost:9200/_ingest/pipeline/enrich-dict-detail-id-policy-pipeline
        删除 enrich
            DELETE http://localhost:9200/_enrich/policy/enrich-dict-detail-id-policy
