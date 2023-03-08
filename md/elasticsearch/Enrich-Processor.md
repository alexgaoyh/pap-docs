# Elasticsearch ���� Enrich Processor ʵ�� MYSQL Join �Ĳ�����֧��Nested���ͣ�����Ӧ�á�
    �ڹ�ϵ�����ݿ��У�������Ƿǳ������Ĳ������ŵ� Elasticsearch �ĳ������൱�ڿ������������ݹ���������ʹ�� Enrich Processor��������Ԥ����׶ΰ����ݽ���ά��������ʹ�������ڴ洢��ʱ�򣬾����Ѿ����й����ݹ����ġ�
    
    ��һ���ܳ����� ʡ���������������������(ͬ�������� �ֵ� ����ĳ���)���ڱ������ݵ�ʱ�򣬰ѹ��������ݲ�ѯ�������浽 document �С�

    1����ʼ����
        �����ʼ��һ�������� dict-detail�������а���������ֵ������ʾ������ keyword ���ͣ�����ֵ������ʾ��
    
            public DictDetail(String dictDetailId, String dictId, String dictCode, String dictName, String dictDetailCode, String dictDetailName) {
                this.dictDetailId = dictDetailId;
                this.dictId = dictId;
                this.dictCode = dictCode;
                this.dictName = dictName;
                this.dictDetailCode = dictDetailCode;
                this.dictDetailName = dictDetailName;
            }
    
            DictDetail dictDetail2 = new DictDetail("410102", "410100", "410100", "֣����", "410102", "��ԭ��");
            DictDetail dictDetail3 = new DictDetail("410103", "410100", "410100", "֣����", "410103", "������");
            DictDetail dictDetail4 = new DictDetail("410104", "410100", "410100", "֣����", "410104", "�ܳǻ�����");
            DictDetail dictDetail5 = new DictDetail("410105", "410100", "410100", "֣����", "410105", "��ˮ��");
            DictDetail dictDetail6 = new DictDetail("410106", "410100", "410100", "֣����", "410106", "�Ͻ���");
            DictDetail dictDetail8 = new DictDetail("410108", "410100", "410100", "֣����", "410108", "�ݼ���");

    2��ִ��Ч��չʾ����Ҫ����ִ�� 3.1��3.2��3.3 ���ֵ������
        2.1���ĵ�������
            PUT http://localhost:9200/test-dict-detail/_doc/1?pipeline=enrich-dict-detail-id-policy-pipeline
                {
                    "dictDetailId": "410102",
                    "comment": "ϣ������֮���ܹ�����dictDetailId��Ӧ����չ�ֵ���ϢdictDetailId_enriched",
                    "address": [
                        {
                            "dictDetailId": "410102"
                        },
                        {
                            "dictDetailId": "410103"
                        }
                    ]
                }

        2.2���ĵ��鿴�� ע�Ᵽ�浽�����е��ĵ��������������ԣ�һ���� dictDetailId ��չ������ dictDetailId_enriched��  ��һ���� address.dictDetailId ��չ������ address._dictDetailIdObj

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
                                    "dictName": "֣����",
                                    "dictId": "410100",
                                    "dictDetailName": "��ԭ��",
                                    "dictCode": "410100",
                                    "dictDetailCode": "410102"
                                },
                                "dictDetailId": "410102"
                            },
                            {
                                "_dictDetailIdObj": {
                                    "dictDetailId": "410103",
                                    "dictName": "֣����",
                                    "dictId": "410100",
                                    "dictDetailName": "������",
                                    "dictCode": "410100",
                                    "dictDetailCode": "410103"
                                },
                                "dictDetailId": "410103"
                            }
                        ],
                        "dictDetailId": "410102",
                        "comment": "ϣ������֮���ܹ�����dictDetailId��Ӧ����չ�ֵ���ϢdictDetailId_enriched",
                        "dictDetailId_enriched": {
                            "dictDetailId": "410102",
                            "dictName": "֣����",
                            "dictId": "410100",
                            "dictDetailName": "��ԭ��",
                            "dictCode": "410100",
                            "dictDetailCode": "410102"
                        }
                    }
                }


    3��ʵ�ַ�����
        ��� enrich processor �ĸ���ܶ��ĵ������������������Ľ�ϴ��������ϸ˵����
        
        3.1���������ԣ��ò��Զ������ǽ�ʹ���ĸ��ֶν��������������������е��ĵ�����ƥ�䡣

            match �� policy ���ͣ����˴�ͳ��match���ͣ�����Ӧ���ڵ���λ�ó����ģ�geo_match��
            indices : һ������Դ�������б��洢���Ǵ� enrich ��չ�����ݡ�
            match_field �� Դ����������ƥ�䴫���ĵ���ƥ���ֶΡ�
            enrich_field �� Դ�����е��ֶ��б�������ӵ��´�����ĵ��С�

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
        
        3.2�����в���
            PUT http://localhost:9200/_enrich/policy/enrich-dict-detail-id-policy/_execute

        3.3������һ�� pipeline
            ע�����������ͨfield���Ժ�Nested-Object���Զ�������������
            
            3.3.1��processors �еĵ�һ�� enrich ������� ��ͨfield���ԣ�ֻҪƥ�䵽 dictDetailId ���ԣ��Ͷ����������� dictDetailId_enriched�� ������� 2.2 ���ֵĽ����
            3.3.2��processors �е�ʣ��Ĳ���������� Nested-Object���еĲ�����ֻҪƥ�䵽 address.dictDetailId���Ͷ����������� address._dictDetailIdObj��������� 2.2 ���ֵĽ����
                3.3.2.1������ʹ�� foreach ���� address ���ԣ�ȡ�������е� dictDetailId���ԣ����� addressDictDetailIds ���飻
                3.3.2.2��ʹ�õ�һ�����ɵ� addressDictDetailIds ���飬������������ addressDictDetailIds_enriched �����������������������Ѿ�������������Ķ���
                3.3.2.3��ɾ�� addressDictDetailIds ���飻
                3.3.2.4���� addressDictDetailIds_enriched ������� �� ԭʼ�� address ���������кϲ����� address ������������� _dictDetailIdObj ���ԣ�
                3.3.2.5��ɾ�� addressDictDetailIds_enriched �������

            PUT http://localhost:9200/_ingest/pipeline/enrich-dict-detail-id-policy-pipeline

            {
                "description": "Enrich dict-detail-id information",
                "processors": [
                    {
                        "enrich": {
                            "description": "����������� field ���Խ�������,���� target_field ����",
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
                            "description": "Nested-Object-Enrich:1������ field ����, ���� field ����",
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
                            "description": "Nested-Object-Enrich:2����Ե�һ�����ɵ����飬������������ target_field �������",
                            "policy_name": "enrich-dict-detail-id-policy",
                            "field": "addressDictDetailIds",
                            "target_field": "addressDictDetailIds_enriched",
                            "max_matches": "128",
                            "ignore_failure": true
                        }
                    },
                    {
                        "remove": {
                            "description": "Nested-Object-Enrich:3����Ե�һ�����ɵ����飬�Ѿ������˵ڶ����Ĵ�����ɾ����һ����ʱ���ɵ��������",
                            "ignore_failure": true,
                            "field": [
                                "addressDictDetailIds"
                            ]
                        }
                    },
                    {
                        "script": {
                            "description": "Nested-Object-Enrich:4����ԭʼ����͵�3�����ɵ����������кϲ�",
                            "lang": "painless",
                            "ignore_failure": true,
                            "source": "for (int i = 0; i < ctx.address.length; i++) { ctx.address[i]._dictDetailIdObj = ctx.addressDictDetailIds_enriched[i] }"
                        }
                    },
                    {
                        "remove": {
                            "description": "Nested-Object-Enrich:5��ɾ���ڶ����������ɵĶ���",
                            "field": [
                                "addressDictDetailIds_enriched"
                            ],
                            "ignore_failure": true
                        }
                    }
                ]
            }
    
        3.4���Ϳ���ʹ�� 2.1 �� 2.2 �������ֵ���������ĵ��Ͳ鿴�ĵ����������� Mysql �� join �������Ѿ�ʵ�֣�

    4���������
        ɾ�� pipeline
            DELETE http://localhost:9200/_ingest/pipeline/enrich-dict-detail-id-policy-pipeline
        ɾ�� enrich
            DELETE http://localhost:9200/_enrich/policy/enrich-dict-detail-id-policy
