# dynamic_templates
    前些年在使用 Elasticsearch 的时候，看到过 动态模板(Dynamic templates) 相关的知识点，但并没有想到如何在实际业务中应用，最近又看到这个知识点，
    结合前些年被广泛提及的"低代码平台"，突然意识到如果有一个很简单的增删改查需求，前端通过推拽的方式组成页面，后端如果使用 Elasticsearch 进行数据存储，
    直接将前端传递过来的 JSON 数据进行存储，至此需求就可以完成。 但是存在的问题是数据类型需要调整，便于后续的查询和统计，那么 动态模板(Dynamic templates) 
    就可以解决这个问题，在创建索引的时候，约定好数据类型，后续根据约定生成不同的数据类型。

    动态模板(Dynamic templates)可以在创建mapping时，先定义好规则，当新字段满足某条规则时，就会按照该规则的预先配置来创建字段。

    如下定义了一个较为通用的 dynamic_templates  配置，其中约定了 integer long date boolean scaled_float 等数据类型(string_to_*)，并且针对 string 类型的数据，修改了默认的分词器规则为 IK 中文分词器。

        "date_detection": false 的配置，关闭了日期格式的自动检测，避免识别错误， 形如 "createDate": "2000-01-01" 的格式，会不能按照如下的格式进行识别。
        string_to_default_ik_string 部分，写在最后部分说明在当前的配置中优先级最低，并且修改了剩余的 字符串 类型的分词方式(IK)。
        object_list_to_nested 部分，如果是以 List 结尾的话，约定 "type": "nested" ，避免数据被平铺。 

    PUT http://localhost:9200/${indexName}

    {
        "mappings": {
            "date_detection": false,
            "dynamic_templates": [
                {
                    "string_to_integer": {
                        "match_mapping_type": "string",
                        "match": "*Int",
                        "mapping": {
                            "type": "integer"
                        }
                    }
                },
                {
                    "string_as_num": {
                        "match_mapping_type": "string",
                        "match": "*Num",
                        "mapping": {
                            "type": "integer"
                        }
                    }
                },
                {
                    "string_to_long": {
                        "match_mapping_type": "string",
                        "match": "*Long",
                        "mapping": {
                            "type": "long"
                        }
                    }
                },
                {
                    "string_to_date": {
                        "match_mapping_type": "string",
                        "match": "*Date",
                        "mapping": {
                            "type": "keyword",
                            "fields": {
                                "date": {
                                    "type": "date",
                                    "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
                                }
                            }
                        }
                    }
                },
                {
                    "string_to_time": {
                        "match_mapping_type": "string",
                        "match": "*Time",
                        "mapping": {
                            "type": "keyword",
                            "fields": {
                                "date": {
                                    "type": "date",
                                    "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
                                }
                            }
                        }
                    }
                },
                {
                    "string_to_bool": {
                        "match_mapping_type": "string",
                        "match": "*Bool",
                        "mapping": {
                            "type": "boolean"
                        }
                    }
                },
                {
                    "string_to_point_scaled_float": {
                        "match_mapping_type": "string",
                        "match": "*Point",
                        "mapping": {
                            "type": "scaled_float",
                            "scaling_factor": 100
                        }
                    }
                },
                {
                    "string_to_price_scaled_float": {
                        "match_mapping_type": "string",
                        "match": "*Price",
                        "mapping": {
                            "type": "scaled_float",
                            "scaling_factor": 100
                        }
                    }
                },
                {
                    "object_list_to_nested": {
                        "match_mapping_type": "object",
                        "match": "*List",
                        "mapping": {
                            "type": "nested"
                        }
                    }
                },
                {
                    "string_to_default_ik_string": {
                        "match_mapping_type": "string",
                        "match": "*",
                        "mapping": {
                            "type": "text",
                            "analyzer": "ik_max_word",
                            "search_analyzer": "ik_max_word",
                            "fields": {
                                "keyword": {
                                    "type": "keyword"
                                }
                            }
                        }
                    }
                }
            ]
        }
    }
