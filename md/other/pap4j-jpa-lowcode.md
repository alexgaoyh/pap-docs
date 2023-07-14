# [低代码平台、国产化]基于JPA的简易伪低代码模块

## 背景

&ensp;&ensp;一方面鉴于国内外环境的变化，另一方面鉴于软件开发的复杂性，期望构建一款兼容国产环境（特别是国产数据库）的简易低代码模块。

## 现状

&ensp;&ensp;1、各数据库厂商发布的数据库都有自己的特性（语句差异、函数差异、序列查询……），实际编码过程中很难遵守SQL92/SQL99标准，再加上国内中小企业更喜欢使用Mybatis，更难以在编码层面进行控制。

&ensp;&ensp;2、现有的低代码平台为能够快速便捷的解决80%的问题，但是针对特殊的需求，由于代码量大，结构分层多，封装过度，很难快速的进行业务处理。

## 解决思路

&ensp;&ensp;1、采用 JAP 实现数据持久化，借助此ORM规范兼容各个数据库厂商，同时在此基础上引入
QueryDSL，从而支持更复杂的查询业务，提升JPA关联查询的不足。

&ensp;&ensp;2、为了减少代码复杂程度，将传统MVC分层架构中的业务层去除，之所以这样考虑，是基于数据最终还是要持久化到数据库，直接在同一个事务中完成数据的处理即可。

&ensp;&ensp;3、在编写的 JPA实体类 中做文章，采用增加注解的方案，向外提供更友好的字段展示（非必要）。

## 详细方案

### JPA 、QueryDSL 引入

1. 修改pom.xml文件，引入如下依赖： spring-boot-starter-data-jpa、querydsl-apt、querydsl-jpa；
2. 进行整合与处理，配置 JPAQueryFactory 对象；

### 主键生成规则（可选）

&ensp;&ensp;可以根据需求确定是否引入不同的主键生成规则，当前引入了雪花算法作为主键生成策略。

### 动态搜索（建议）

&ensp;&ensp;参考[querydsl-dynamic-query](https://github.com/Abhi-Codes/querydsl-dynamic-query)模块，去完成动态列的搜索功能，提升搜索支持。

&ensp;&ensp;可以按照如下的 Operator操作符 去完成 范围搜索、比较搜索、模糊搜索、等值搜索……。

&ensp;&ensp;如下代码块即可完成动态参数的查询功能。

|  Operator   |         Meaning         | Applicable for Data Type |
|:-----------:|:-----------------------:|:------------------------:|
|      :      |           In            | Integer,Long,Double,Date |
|      >      |      Greater than       | Integer,Long,Double,Date |
|     >=      |  Greater than equal to  | Integer,Long,Double,Date |
|      <      |        Less than        | Integer,Long,Double,Date |
|     <=      |   Less than equal to    | Integer,Long,Double,Date |
|      :      |   Equals Ignore Case    |          String          |
|      %      | Starts With Ignore Case |          String          |
|      -      |  Contains Ignore Case   |          String          |
|     ()      |     Inclusive Range     |           Date           |

```html
List<SearchCriteria> criterias = new ArrayList<SearchCriteria>();
criterias.add(new SearchCriteria("loginName", ":", "alexgaoyh1"));
criterias.add(new SearchCriteria("realName", ":", "alexgaoyh1"));
BooleanExpression build = new CommonPredicateBuilder<>(SysUser.class).and(criterias).build();
List<SysUser> fieldEqualList = jpaQueryFactory.selectFrom(qSysUser).where(build).fetch();
```

### 代码示例（推荐）

&ensp;&ensp;完整代码参考[pap4j-jpa-lowcode](https://gitee.com/alexgaoyh/pap4j-jpa-lowcode)

&ensp;&ensp;部分核心代码详见如下单元测试，分别完成了简单的插入和查询操作、基于QuyerDSL的搜索、扩展QueryDSL的动态列搜索功能。

&ensp;&ensp;单表增删改查功能可参考如上[完整代码](https://gitee.com/alexgaoyh/pap4j-jpa-lowcode)

```java
@SpringBootTest
class Pap4jJpaLowcodeApplicationTests {

    @Autowired
    private SessionFactory sessionFactory;

    @Autowired
    private JPAQueryFactory jpaQueryFactory;

    @Test
    void insertSysUser() {
        Session session = sessionFactory.openSession();
        // save
        Transaction tx = session.beginTransaction();
        SysUser sysUser = new SysUser();
        sysUser.setLoginName("alexgaoyh");
        Serializable save = session.save(SysUser.class.getName(), sysUser);
        tx.commit();

        // query
        Query query = session.createQuery("from " + SysUser.class.getName());
        List list = query.getResultList();
        System.out.println(list.toString());

        session.close();
    }

    @Test
    void queryDSLSysUser() {
        QSysUser qSysUser = QSysUser.sysUser;
        // 查询所有
        List<SysUser> allList = jpaQueryFactory.selectFrom(qSysUser).fetch();
        // 单条件查询
        SysUser oneConditionSysUser = jpaQueryFactory.selectFrom(qSysUser).where(qSysUser.loginName.eq("alexgaoyh1")).fetchOne();
        // 多条件查询
        SysUser multiConditionSysUser = jpaQueryFactory.selectFrom(qSysUser).where(qSysUser.loginName.eq("alexgaoyh2").and(qSysUser.realName.eq("alexgaoyh2"))).fetchOne();
        // 排序查询
        List<SysUser> orderBySysUser = jpaQueryFactory.selectFrom(qSysUser).orderBy(qSysUser.sysUserId.desc()).fetch();
        // 分页条件查询
        List<SysUser> pageSysUser = jpaQueryFactory.selectFrom(qSysUser).where(qSysUser.loginName.like("alexgaoyh%")).orderBy(qSysUser.sysUserId.desc()).offset(0).limit(10).fetch();
        System.out.println(pageSysUser);
    }

    @Test
    void commonPredicateSysUser() {
        Session session = sessionFactory.openSession();
        // save
        Transaction tx = session.beginTransaction();
        for (int i = 0; i < 20; i++) {
            SysUser sysUser = new SysUser();
            sysUser.setLoginName("alexgaoyh" + i);
            sysUser.setRealName("alexgaoyh" + i);
            Serializable save = session.save(SysUser.class.getName(), sysUser);
        }
        tx.commit();

        QSysUser qSysUser = QSysUser.sysUser;
        List<SearchCriteria> criterias = new ArrayList<SearchCriteria>();
        criterias.add(new SearchCriteria("loginName", ":", "alexgaoyh1"));
        criterias.add(new SearchCriteria("realName", ":", "alexgaoyh1"));
        BooleanExpression build = new CommonPredicateBuilder<>(SysUser.class).and(criterias).build();
        List<SysUser> fieldEqualList = jpaQueryFactory.selectFrom(qSysUser).where(build).fetch();
        System.out.println(fieldEqualList);
    }
}
```

## 参考

1. https://gitee.com/alexgaoyh/pap4j-jpa-lowcode 
2. https://github.com/Abhi-Codes/querydsl-dynamic-query
3. https://blog.csdn.net/footless_bird/article/details/129359394
4. https://juejin.cn/post/6863303441607983111
