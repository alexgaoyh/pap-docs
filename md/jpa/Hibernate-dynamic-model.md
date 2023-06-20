### Hibernate dynamic model 动态模型

## 介绍

&ensp;&ensp;Hibernate的动态模型为我们动态改动表结构带来了方便, 个人认为这一点非常有价值, 现在的企业级应用系统越来越强调用户可定制性, hibernate的这一点使用户自定义字段或自定义表成为可能 .

## 场景

1. 启动时注册
    1. 使用 resources/hibernate/*.hbm.xml 文件的方式，在启动过程中，自动将类注册到 SessionFactory 中；
    2. 使用 Query query = session.createQuery("from DynamicTableColumn where tableName = '" + tableName + "'"); 进行查询。

2. 运行时注册
    1. 容器启动完成后，程序正常运行期间产生的类，Hibernate 并不认识，所以需要进行注册；
    2. Hibernate 官方建议sessionFactory一旦创建好了，就不要对其做修改，所以如何对新创建的类进行管理和使用就很重要；
    3. 使用 Spring 提供的 LocalSessionFactoryBean 来得到SessionFactory;
    4. 详见 LocSessFacBeanConstant.java 类
        1. 定义了 locSessFacBeanList 对象用来存储运行时产生的各个不同的 SessionFactory；
        2. 定义了 get() 方法，用来获取 SessionFactory（支持操作启动时产生的类，也支持运行时产生的类）；

```java
package com.pap.jpa.constant;

import org.hibernate.SessionFactory;
import org.hibernate.boot.Metadata;
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.cfg.Configuration;
import org.hibernate.id.IdentifierGenerator;
import org.hibernate.internal.SessionFactoryImpl;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.tool.hbm2ddl.SchemaUpdate;
import org.hibernate.tool.schema.TargetType;
import org.springframework.orm.hibernate5.LocalSessionFactoryBean;

import javax.persistence.EntityManager;
import javax.sql.DataSource;
import java.io.ByteArrayInputStream;
import java.util.*;

/**
 * LocalSessionFactoryBean
 *
 *
 */
public class LocSessFacBeanConstant {

    /**
     * 存储动态添加的 LocalSessionFactoryBean 信息， 每一个动态模型对应一个 Map。
     */
    public static final List<Map<String, SessionFactory>> locSessFacBeanList = new ArrayList<Map<String, SessionFactory>>();

    /**
     * 获取 SessionFactory 
     * @param entityManager
     * @param dataSource
     * @param entityName
     * @param XML_MAPPING
     * @return
     * @throws Exception
     */
    public static SessionFactory get(EntityManager entityManager, DataSource dataSource, String entityName, String XML_MAPPING) throws Exception {
        SessionFactory sessionFactory = entityManager.getEntityManagerFactory().unwrap(SessionFactory.class);
//        ClassMetadata classMetadata = sessionFactory.getClassMetadata(entityName);
//        String[] propertyNames = classMetadata.getPropertyNames();

        IdentifierGenerator identifierGenerator = ((SessionFactoryImpl) sessionFactory).getIdentifierGenerator(entityName);
        if(identifierGenerator != null) {
            return sessionFactory;
        } else {
            for(Map<String, SessionFactory> localSessionFactoryBeanMap : LocSessFacBeanConstant.locSessFacBeanList) {
                if(localSessionFactoryBeanMap.containsKey(entityName)) {
                    return localSessionFactoryBeanMap.get(entityName);
                }
            }

            LocalSessionFactoryBean localSessionFactoryBean = new LocalSessionFactoryBean();
            localSessionFactoryBean.setDataSource(dataSource);
            localSessionFactoryBean.afterPropertiesSet();

            Configuration configuration = localSessionFactoryBean.getConfiguration();
            ServiceRegistry serviceRegistry = new StandardServiceRegistryBuilder().applySettings(configuration.getProperties()).build();

            MetadataSources metadataSources = new MetadataSources(serviceRegistry);
            //读取映射文件
            metadataSources.addInputStream(new ByteArrayInputStream(XML_MAPPING.getBytes()));
            Metadata metadata = metadataSources.buildMetadata();
            //更新数据库Schema,如果不存在就创建表,存在就更新字段,不会影响已有数据
            SchemaUpdate schemaUpdate = new SchemaUpdate();
            schemaUpdate.execute(EnumSet.of(TargetType.DATABASE), metadata, serviceRegistry);

            //合并配置
            configuration.addInputStream(new ByteArrayInputStream(XML_MAPPING.getBytes()));
            SessionFactory newSessionFactory = configuration.buildSessionFactory(serviceRegistry);

            Map<String, SessionFactory> tmp = new HashMap<>();
            tmp.put(entityName, newSessionFactory);
            locSessFacBeanList.add(tmp);

            return newSessionFactory;
        }

    }
}
```

## 测试

&ensp;&ensp;如 TestController 所示，假设在运行时定义了 XML_MAPPING 对象，根据 test() 方法，首先获取 SessionFactory 对象，并进行插入和查询操作

1. LocSessFacBeanConstant.get() 方法，如果启动时产生的 SessionFactory 是否包含 Student 对象，不包含进入步骤2，否则直接返回 SessionFactory；
2. 判断 LocSessFacBeanConstant.locSessFacBeanList 中是否已经生成了 Student 对象对应的 SessionFactory， 如果没有生成进入步骤3，否则直接返回 locSessFacBeanList 内指定的对象；
3. 定义 LocalSessionFactoryBean 对象，并生成相关表结构，对配置进行合并，并将生成的 newSessionFactory 加入到 locSessFacBeanList 对象中，同时返回；

详见如下 TestController.java 文件。

```java
package com.pap.jpa.controller;

import com.pap.jpa.constant.LocSessFacBeanConstant;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.persistence.EntityManager;
import javax.persistence.Query;
import javax.servlet.http.HttpServletRequest;
import javax.sql.DataSource;
import java.util.*;
import java.util.List;
import java.util.Map;

@RestController
public class TestController {

    @Autowired
    private EntityManager entityManager;

    @Autowired
    private DataSource dataSource;

    /**
     * 运行期的持久化实体没有必要一定表示为像POJO类或JavaBean对象那样的形式。
     * Hibernate也支持动态模型在运行期使用Map和象DOM4J的树模型那样的实体表示。
     * 使用这种方法，你不用写持久化类，只写映射文件就行了。
     **/
    public static final String XML_MAPPING = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n" +
            "<!DOCTYPE hibernate-mapping PUBLIC\n" +
            "        \"-//Hibernate/Hibernate Mapping DTD 3.0//EN\"\n" +
            "        \"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd\">\n" +
            "<hibernate-mapping>\n" +
            "    <class entity-name=\"Student\" table=\"t_student\">\n" +
            "        <id name=\"id\" type=\"java.lang.Long\" length=\"64\" unsaved-value=\"null\">\n" +
            "            <generator class=\"identity\" />\n" +
            "        </id>" +
            "        <property type=\"java.lang.String\" name=\"userName\" column=\"userName\"/>\n" +
            "        <property name=\"age\" type=\"java.lang.Integer\" column=\"age\"/>\n" +
            "    </class>" +
            "</hibernate-mapping>";


    @GetMapping("/test")
    public void test(HttpServletRequest request) {
        try {
            SessionFactory newSessionFactory = LocSessFacBeanConstant.get(entityManager, dataSource, "Student", XML_MAPPING);
            //保存对象
            Session newSession = newSessionFactory.openSession();

            Map<String, Object> student = new HashMap<>();
            student.put("userName", "alexgaoyh");
            student.put("sex", "male");
            newSession.save("Student", student);
            
            //查询所有对象
            Query query = newSession.createQuery("from Student");
            List list = query.getResultList();

            newSession.close();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

## 依赖
1. spring-boot-starter-data-jpa  2.1.8.RELEASE
2. spring-boot-starter-web 2.1.8.RELEASE

## 参考
1. https://www.jianshu.com/p/9fe01c47d3ff
