# copy_to 与 multi_match

    1、举例，将 first_name 和 last_name 这两个字段的值都会复制到full_name上，然后我们在查询的时候，就可以指定从full_name这个字段查询了，可以简化查询命令

    {
        "mappings": {
            "properties": {
                "first_name": {
                    "type": "text",
                    "copy_to": "full_name"
                },
                "last_name": {
                    "type": "text",
                    "copy_to": "full_name"
                },
                "full_name": {
                    "type": "text"
                }
            }
        }
    }

    2、在使用 multi-match 的过程中，有三种类型： 
        best_fields(多个字段中，返回评分最高的，类似 dix_max)
        most_fields(匹配多个字段，返回的综合评分（非最高分），类似 bool should )
        cross_fields(跨字段匹配，待查询内容在多个字段中都显示，希望在任何这些列出的字段中尽可能找出多的词)(它首先将查询字符串分词，然后在任何字段中查找每个分词，就好像它们是一个大字段。)

    {
        "query": {
            "multi_match": {
                "query": "分词效果",
                "type": "cross_fields",
                "operator": "and",
                "fields": [
                    "studentName",
                    "studentIntro^10"
                ]
            }
        }
    }
