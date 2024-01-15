## 背景

&ensp;&ensp;《替代关系型数据库 MAX 聚合函数的思路》，在报表数据查询的场景中，有时候会需要在类似日志表中查询指定条件下的最新数据，一般的思路是先通过 select max(id) from table group by bus_key 查询到最新的数据，然后外部再包装一个查询语句。本文提供自关联查询的方式去提供另一种思路。

## 思路

&ensp;&ensp;针对从类似日志表中查询最新的一条数据，可以使用自关联查询的方式，直接把最新的数据查询出来，避免使用 max 函数。 本思路并不保证可以提升查询的执行效率。

## 相关SQL

&ensp;&ensp;提供了两组查询语句，实现的是同样的业务逻辑，唯一的区别在于多了一个过滤条件，相当于多个不同的分组，具体实现可以看下属语句。

&ensp;&ensp;示例相当于有一张学生成绩的记录表，主键ID是递增的，分别记录学生的编号，所属学年，当前的成绩和录入时间。想要查询出来每个学生的最后一次成绩。 第二组SQL查询语句是多了所属学校的过滤，返回的是不同学校不同学生编号的最新一次成绩。

```mysql
CREATE TABLE `score` (
`ID` int(11) NOT NULL COMMENT '递增主键',
`STUDENT_ID` int(11) DEFAULT NULL COMMENT '学生编号',
`SCHOOL_YEAR` int(11) DEFAULT NULL COMMENT '学年',
`SCORE` double(5,2) DEFAULT NULL COMMENT '成绩',
`CREATE_TIME` datetime DEFAULT NULL COMMENT '成绩录入时间',
PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;


SELECT DISTINCT t1.*
FROM score t1
LEFT JOIN score t2 ON t1.STUDENT_ID = t2.STUDENT_ID AND t1.ID < t2.ID
WHERE t2.ID IS NULL
ORDER BY CREATE_TIME ASC, STUDENT_ID ASC;
```

```mysql

CREATE TABLE `score2` (
`ID` int(11) NOT NULL COMMENT '递增主键',
`STUDENT_ID` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '学生编号',
`SCHOOL_NAME` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin DEFAULT NULL COMMENT '所属学校名称',
`SCHOOL_YEAR` int(11) DEFAULT NULL COMMENT '学年',
`SCORE` double(5,2) DEFAULT NULL COMMENT '成绩',
`CREATE_TIME` datetime DEFAULT NULL COMMENT '成绩录入时间',
PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;


SELECT DISTINCT t1.*
FROM score2 t1
LEFT JOIN score2 t2 ON t1.STUDENT_ID = t2.STUDENT_ID
AND t1.SCHOOL_NAME = t2.SCHOOL_NAME AND t1.ID < t2.ID
WHERE t2.ID IS NULL
ORDER BY CREATE_TIME ASC, STUDENT_ID ASC;
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
