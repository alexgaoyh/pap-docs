# ElasticSearch 高级检索，按照顺序进行搜索。

&ensp;&ensp;在数据检索的应用场景中，经常出现高级检索的功能需求，指定不同的字段与不同的逻辑关系，对数据进行检索。在某些应用场景中，会要求按照检索条件的顺序进行数据查找。

&ensp;&ensp;为解决按照检索添加出现的顺序进行检索的需求，本文使用 Elasticsearch 的 DSL 进行数据测试。

&ensp;&ensp;按照高级检索中检索条件出现的顺序进行条件处理，并约定如果两个检索条件之间选择‘与’关系，则取交集， 如果两个检索条件之间选择‘或’关系，则取并集。

&ensp;&ensp;下面优先初始化信息，并根据不同的检索条件，编写对应的 DSL 脚本，并同时满足上述约定。

## 数据初始化
```html
    public Student(String id, String studentName, String studentNo, Integer studentAge, String studentBirth, String studentIntro, String studentTags, BigDecimal gradePoint) {
        this.id = id;
        this.studentName = studentName;
        this.studentNo = studentNo;
        this.studentAge = studentAge;
        this.studentBirth = studentBirth;
        this.studentIntro = studentIntro;
        this.studentTags = studentTags;
        this.gradePoint = gradePoint;
    }

    Student student28 = new Student("28", "酒精", "1234567890", 80, StringUtilss.now(), "酒精", "杀毒、消菌", new BigDecimal("10"));
    studentRepository.save(student28);
    
    Student student29 = new Student("29", "酒精", "1234567890", 81, StringUtilss.now(), "酒精", "杀毒、消菌", new BigDecimal("11"));
    studentRepository.save(student29);
    
    Student student30 = new Student("30", "酒精", "1234567890", 82, StringUtilss.now(), "酒精", "杀毒、消菌", new BigDecimal("12"));
    studentRepository.save(student30);
    
    Student student31 = new Student("31", "白酒", "1234567890", 90, StringUtilss.now(), "白酒", "饮用、聚餐", new BigDecimal("20"));
    studentRepository.save(student31);
    
    Student student32 = new Student("32", "白酒", "1234567890", 91, StringUtilss.now(), "白酒", "饮用、聚餐", new BigDecimal("21"));
    studentRepository.save(student32);
    
    Student student33 = new Student("33", "白酒", "1234567890", 92, StringUtilss.now(), "白酒", "饮用、聚餐", new BigDecimal("22"));
    studentRepository.save(student33);
```

## 条件：A与B
&ensp;&ensp;理解为将满足条件 A 的结果与满足条件 B 的结果求交集。

&ensp;&ensp;举一个实际的场景，假设条件为： (studentName = 酒精) 与 (studentName = 白酒) ，那么将没有任何数据进行返回。

&ensp;&ensp;使用如下命令进行数据查找，返回 0 条数据。 hits.total.value = 0
```json
{"size":0,"query":{"bool":{"must":[{"bool":{"must":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}}}
```

## 条件：A或B
&ensp;&ensp;理解为将满足条件 A 的结果和满足条件 B 的结果求并集。

&ensp;&ensp;举一个实际的场景，假设条件为： (studentName = 酒精) 或 (studentName = 白酒) ，那么将返回如上6条数据。

&ensp;&ensp;使用如下命令进行数据查找，返回 6 条数据。 hits.total.value = 6
```json
{"size":0,"query":{"bool":{"must":[{"bool":{"should":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}}}
```

## 条件：(A与B)或C
&ensp;&ensp;理解为将满足条件 A 的结果和满足条件 B 的结果求交集，之后再与条件 C 求并集。

&ensp;&ensp;举一个实际的场景，假设条件为： ((studentName = 酒精) 与 (studentName = 白酒)) 或 (studentAge in [ 80, 90 ]) ，那么将返回如上4条数据。

&ensp;&ensp;使用如下命令进行数据查找，返回 4 条数据。 hits.total.value = 4
```json
{"size":0,"query":{"bool":{"should":[{"bool":{"must":[{"bool":{"must":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}},{"range":{"studentAge":{"gte":80,"lte":90}}}]}}}
```

## 条件：(A与B)与C
&ensp;&ensp;理解为将满足条件 A 的结果和满足条件 B 的结果求交集，之后再与条件 C 求交集。

&ensp;&ensp;举一个实际的场景，假设条件为： ((studentName = 酒精) 与 (studentName = 白酒)) 与 (studentAge in [ 80, 90 ]) ，那么将返回如上0条数据。

&ensp;&ensp;使用如下命令进行数据查找，返回 0 条数据。 hits.total.value = 0
```json
{"size":0,"query":{"bool":{"must":[{"bool":{"must":[{"bool":{"must":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}},{"range":{"studentAge":{"gte":80,"lte":90}}}]}}}
```

## 条件：(A或B)或C
&ensp;&ensp;理解为将满足条件 A 的结果和满足条件 B 的结果求并集，之后再与条件 C 求并集。

&ensp;&ensp;举一个实际的场景，假设条件为： ((studentName = 酒精) 并 (studentName = 白酒)) 并 (studentAge in [ 80, 90 ]) ，那么将返回如上6条数据。

&ensp;&ensp;使用如下命令进行数据查找，返回 6 条数据。 hits.total.value = 6
```json
{"size":0,"query":{"bool":{"should":[{"bool":{"must":[{"bool":{"should":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}},{"range":{"studentAge":{"gte":80,"lte":90}}}]}}}
```

## 条件：(A或B)与C
&ensp;&ensp;理解为将满足条件 A 的结果和满足条件 B 的结果求并集，之后再与条件 C 求交集。

&ensp;&ensp;举一个实际的场景，假设条件为： ((studentName = 酒精) 并 (studentName = 白酒)) 与 (studentAge in [ 80, 90 ]) ，那么将返回如上4条数据。

&ensp;&ensp;使用如下命令进行数据查找，返回 4 条数据。 hits.total.value = 4
```json
{"size":0,"query":{"bool":{"must":[{"bool":{"must":[{"bool":{"should":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}},{"range":{"studentAge":{"gte":80,"lte":90}}}]}}}
```

## 条件：((A与B)或C)与D
&ensp;&ensp;理解为将满足条件 A 的结果和满足条件 B 的结果求交集，之后再与条件 C 求并集，最后再与条件 D 求交集。

&ensp;&ensp;举一个实际的场景，假设条件为： (((studentName = 酒精) 与 (studentName = 白酒)) 并 (studentAge in [ 80, 90 ])) 与 (studentTags = 饮用、聚餐) ，那么将返回如上1条数据。

&ensp;&ensp;使用如下命令进行数据查找，返回 1 条数据。 hits.total.value = 1
```json
{"size":0,"query":{"bool":{"must":[{"bool":{"should":[{"bool":{"must":[{"bool":{"must":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}},{"range":{"studentAge":{"gte":80,"lte":90}}}]}},{"match":{"studentTags":{"query":"饮用、聚餐"}}}]}}}
```

## 条件：((A与B)或C)或D
&ensp;&ensp;理解为将满足条件 A 的结果和满足条件 B 的结果求交集，之后再与条件 C 求并集，最后再与条件 D 求并集。

&ensp;&ensp;举一个实际的场景，假设条件为： (((studentName = 酒精) 与 (studentName = 白酒)) 并 (studentAge in [ 80, 90 ])) 并 (studentTags = 饮用、聚餐) ，那么将返回如上6条数据。

&ensp;&ensp;使用如下命令进行数据查找，返回 6 条数据。 hits.total.value = 6
```json
{"size":0,"query":{"bool":{"should":[{"bool":{"should":[{"bool":{"must":[{"bool":{"must":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}},{"range":{"studentAge":{"gte":80,"lte":90}}}]}},{"match":{"studentTags":{"query":"饮用、聚餐"}}}]}}}
```

## 条件：(((A与B)或C)与D)与E
&ensp;&ensp;理解为将满足条件 A 的结果和满足条件 B 的结果求交集，之后再与条件 C 求并集，接下来再与条件 D 求交集，最后再与条件 E 求交集。

&ensp;&ensp;举一个实际的场景，假设条件为： ((((studentName = 酒精) 与 (studentName = 白酒)) 并 (studentAge in [ 80, 90 ])) 与 (studentTags = 饮用、聚餐) ) 与 (gradePoint = 10)，那么将返回如上1条数据。

&ensp;&ensp;使用如下命令进行数据查找，返回 1 条数据。 hits.total.value = 1
```json
{"size":0,"query":{"bool":{"must":[{"bool":{"must":[{"bool":{"should":[{"bool":{"must":[{"bool":{"must":[{"match":{"studentName":{"query":"酒精"}}},{"match":{"studentName":{"query":"白酒"}}}]}}]}},{"range":{"studentAge":{"gte":80,"lte":90}}}]}},{"match":{"studentTags":{"query":"杀毒、消菌"}}}]}},{"match":{"gradePoint":{"query":"10"}}}]}}}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
