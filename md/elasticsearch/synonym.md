
# ElasticSearch Synonym 同义词

    1、创建同义词文本维护同义词，在服务器上定义一个文本文件，将同义词维护在文本文件中，这样每次需要增删改同义词时，只需要将文本中的内容修改一下即可；
        并且定义 索引模板，如下所示，这样索引 student开头的，并且字段为 studentIntro 的使用了同义词词典；
        
        synonyms.txt
            西红柿,番茄,tomato,圣女果,樱桃
            马铃薯,土豆
        
        PUT http://localhost:9200/_template/template_all
            {
                "template": "student*",
                "settings": {
                    "analysis": {
                        "filter": {
                            "my_local_file_synonym_filter": {
                                "type": "synonym",
                                "synonyms_path": "synonyms.txt"
                            }
                        },
                        "analyzer": {
                            "my_local_file_synonym": {
                                "tokenizer": "ik_smart",
                                "filter": [
                                    "my_local_file_synonym_filter"
                                ]
                            }
                        }
                    }
                },
                "mappings": {
                    "properties": {
                        "studentIntro": {
                            "type": "text",
                            "analyzer": "ik_max_word",
                            "search_analyzer": "my_local_file_synonym"
                        }
                    }
                }
            }
    
    2、如上所示的同义词典为本地的同义词词典，无法完成热加载功能，所以可以使用如下 plugin，将源码下载下来编码后拷贝到ES安装目录下的plugins/dynamic-synonym目录下解压即可；

        https://github.com/bells/elasticsearch-analysis-dynamic-synonym

        使用dynamic-synonym插件的情况下，如果采用服务器文件的方式，当服务器文件的内容发生变化时，则会触发自动更新；插件会读取文件的更新时间来判断是否需要进行同义词更新；
        同样更改 索引模板，增加 studentIntro2 对应的字段
    
        PUT http://localhost:9200/_template/template_all
            {
                "template": "student*",
                "settings": {
                    "analysis": {
                        "filter": {
                            "my_local_file_synonym_filter": {
                                "type": "synonym",
                                "synonyms_path": "synonyms.txt"
                            },
                            "my_remote_file_synonym_filter": {
                                "type": "dynamic_synonym",
                                "synonyms_path": "http://127.0.0.1:81/synonyms.txt",
                                "interval": 30
                            }
                        },
                        "analyzer": {
                            "my_local_file_synonym": {
                                "tokenizer": "ik_smart",
                                "filter": [
                                    "my_local_file_synonym_filter"
                                ]
                            },
                            "my_remote_file_synonym": {
                                "tokenizer": "ik_smart",
                                "filter": [
                                    "my_remote_file_synonym_filter"
                                ]
                            }
                        }
                    }
                },
                "mappings": {
                    "properties": {
                        "studentIntro": {
                            "type": "text",
                            "analyzer": "ik_max_word",
                            "search_analyzer": "my_local_file_synonym"
                        },
                        "studentIntro2": {
                            "type": "text",
                            "analyzer": "ik_max_word",
                            "search_analyzer": "my_remote_file_synonym"
                        }
                    }
                }
            }

    3、可以对如上插件进行修改，增加数据库的读取，但是觉得这样操作增加了 ES 的负担，故这里不再使用此方法，不对其进行额外描述；

    4、增加同义词之后，牵扯到一个 同义词 的搜索结果排序问题，同义词的搜索权重应小于原词，保证原词的召回结果排在前面。
        方法有很多，当前只描述一种方法，使用多字段index ，然后在搜索的时候，不同的字段上设置不同的权重。

        PUT http://localhost:9200/_template/template_all
            {
                "template": "student*",
                "settings": {
                    "analysis": {
                        "filter": {
                            "my_local_file_synonym_filter": {
                                "type": "synonym",
                                "synonyms_path": "synonyms.txt"
                            }
                        },
                        "analyzer": {
                            "my_local_file_synonym": {
                                "tokenizer": "ik_smart",
                                "filter": [
                                    "my_local_file_synonym_filter"
                                ]
                            }
                        }
                    }
                },
                "mappings": {
                    "properties": {
                        "studentIntro": {
                            "type": "text",
                            "analyzer": "ik_max_word",
                            "search_analyzer": "ik_smart",
                            "fields": {
                                "synonym": {
                                    "type": "text",
                                    "analyzer": "ik_max_word",
                                    "search_analyzer": "my_local_file_synonym"
                                }
                            }
                        }
                    }
                }
            }
        
        POST http://localhost:9200/student/_search
            {
                "query": {
                    "bool": {
                        "should": [
                            {
                                "match": {
                                    "studentIntro": {
                                        "query": "西红柿",
                                        "boost": 10
                                    }
                                }
                                },
                                    {
                                    "match": {
                                        "studentIntro.synonym": {
                                        "query": "西红柿",
                                        "boost": 1
                                    }
                                }
                            }
                        ]
                    }
                }
            }

