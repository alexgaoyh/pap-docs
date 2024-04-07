## 概述

&ensp;&ensp;《思考-使用JSON结构映射业务数据与数据库表结构》，本文提供了一种通过JSON结构映射表结构与业务数据关系，从而进行增删改查操作的技术实现。在进行实现之后，发现此实现在真实业务场景中使用的意义并不大，很少有特别匹配的业务场景，故本文不涉及具体实现，只涉及设计思路。

## 介绍

&ensp;&ensp;第一步设计某一个业务场景需要映射到哪几张数据库表结构的哪几个字段，第二步举出此业务场景对应的业务数据结构，第三步对数据解析至可以进行DB操作。

```json lines
// 首先设计业务编码为 C_001_001 的业务场景，此业务场景需要执行 insert 操作，并且需要操作 student、student_ext 这两张表
// 其中 student 表的主键是 student_id， 操作的字段为： student_name、student_age
// 其中 student_ext 表的主键是 student_ext_id，外键是 student_id， 操作的字段为 student_ext_key、student_ext_value
// 通过两个表的主键、外键 的设置，则可以看出来当前业务场景是一个一对多的插入操作
[{
  "papBussId": "C_001_001",
  "operator": "insert",
  "mapping": {
    "student": {
      "pk": "student_id",
      "field": [
        "student_name",
        "student_age"
      ]
    },
    "student_ext": {
      "pk": "student_ext_id",
      "fk": [
        "student_id"
      ],
      "field": [
        "student_ext_key",
        "student_ext_value"
      ]
    }
  }
}]

// 当前业务编码 C_001_001 的调用数据示例如下所示， 提供了一个一对多的数据结构。
{
  "papBussId": "C_001_001",
  "data": [
    {
      "student_name": "pap_alexgaoyh_11",
      "student_age": "pap_1989_11",
      "detail": [
        {
          "student_ext_key": "language",
          "student_ext_value": "english"
        },
        {
          "student_ext_key": "education",
          "student_ext_value": "master"
        }
      ]
    }
  ]
}

// 按照业务映射关系和具体的业务数据，对如上两个JSON结构进行对照映射处理，得到如下的数据结构。
// 数据结构解析完需要对 student、student_ext 两个表进行数据操作
// 首先对 student 进行操作，操作的字段详见 valueMap，其中主键字段是 student_id
// 接着对 student_ext 进行操作，操作的字段详见 valueMap，，其中主键字段是 student_id
[ {
  "tableName" : "student",
  "pk" : "student_id",
  "valueMap" : {
    "student_name" : "pap_alexgaoyh_11",
    "student_age" : "pap_1989_11",
    "student_id" : "0a609bd884554e8698ba2b2ed111a516"
  }
}, {
  "tableName" : "student_ext",
  "pk" : "student_ext_id",
  "valueMap" : {
    "student_ext_key" : "language",
    "student_ext_value" : "english",
    "student_ext_id" : "8bd3f8352b404a84af7499c3961b335e",
    "student_id" : "0a609bd884554e8698ba2b2ed111a516"
  }
}, {
  "tableName" : "student_ext",
  "pk" : "student_ext_id",
  "valueMap" : {
    "student_ext_key" : "education",
    "student_ext_value" : "master",
    "student_ext_id" : "904024e9a65d4b0b9e5e509c2ffd25cb",
    "student_id" : "0a609bd884554e8698ba2b2ed111a516"
  }
} ]
```

## 总结

&ensp;&ensp;按照如上描述，程序设计了一种进行数据插入操作的JSON映射，将业务操作所涉及的表结构字段通过JSON进行描述，同时提供一套业务数据，将业务数据映射为不同的单表操作操作。

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
