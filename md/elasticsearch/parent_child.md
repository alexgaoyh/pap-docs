# ES 父子文档

    详见 Area.java 相关方法与单元测试， 如下是 has_parent has_child 两个命令的使用与测试。    

    curl --location --request POST 'http://localhost:9200/area/_search' \
        --header 'Content-Type: application/json' \
        --data-raw '{
            "query": {
                "has_parent": {
                    "parent_type": "city",
                    "query": {
                        "term": {
                            "areaName.keyword": "许昌市"
                        }
                    }
                }
            }
        }'

    curl --location --request POST 'http://localhost:9200/area/_search' \
        --header 'Content-Type: application/json' \
        --data-raw '{
            "query": {
                "has_child": {
                    "type": "city",
                    "query": {
                        "term": {
                            "areaName.keyword": "许昌市"
                        }
                    }
                }
            }
        }'

    根据 country 分组，取下面 city 下使用 areaName 分组的数据(对每个国家下的城市的名称进行聚合统计).
    {
        "size": 0,
        "aggs": {
            "group_by_country": {
                "terms": {
                    "field": "country.keyword"
                },
                "aggs": {
                    "group_by_child_employee": {
                        "children": {
                            "type": "city"
                        },
                        "aggs": {
                            "group_by_areaName": {
                                "terms": {
                                    "field": "areaName.keyword"
                                }
                            }
                        }
                    }
                }
            }
        }
    }

    包含 area 类型下 areaName = 魏都区 的 province 信息
    {
        "query": {
            "has_child": {
                "type": "city",
                "query": {
                    "has_child": {
                        "type": "area",
                        "query": {
                            "term": {
                                "areaName.keyword": {
                                    "value": "魏都区"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
