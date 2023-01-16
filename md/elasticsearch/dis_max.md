
# ES DIS_MAX : 只是取分数最高的那个 query 的分数而已，完全不考虑其他 query 的分数。
    分离最大化查询（Disjunction Max Query）指的是： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回 ：

    Student student17 = new Student("17", "Keeping pets healthy", "GAO", 22, new Date().toLocaleString(), "My quick brown fox eats rabbits on a regular basis.", "A,B,C,D");
    studentRepository.save(student17);

    Student student18 = new Student("18", "Quick brown rabbits", "GAO", 22, new Date().toLocaleString(), "Brown rabbits are commonly seen.", "A,B,C,D");
    studentRepository.save(student18);

    分别执行如下两个命令， 一个是 bool->should 另一个是 dis_max->queries ，两个命令的结果排序不一致。
        {
            "query": {
                "bool": {
                    "should": [
                        {
                            "match": {
                                "studentName": "Brown fox"
                            }
                        },
                        {
                            "match": {
                                "studentIntro": "Brown fox"
                            }
                        }
                    ]
                }
            }
        }

        {
            "query": {
                "dis_max": {
                    "queries": [
                        {
                            "match": {
                                "studentName": "Brown fox"
                            }
                        },
                        {
                            "match": {
                                "studentIntro": "Brown fox"
                            }
                        }
                    ]
                }
            }
        }
