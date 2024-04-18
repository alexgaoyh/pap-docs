# [图数据库]gStore1.2在Ubuntu和Java环境下的安装与试用

## 背景

&ensp;&ensp;近期在互联网上看到关于gStore这款面向大规模知识图谱应用的原生图数据库系统，本文基于Ubuntu22.04，进行安装和基于Java语言的试用。

## 在Ubuntu22.04版本下的安装步骤

1. apt-get update
2. apt-get install libjemalloc-dev
2. 从GitHub上下载源码		https://github.com/pkumod/gStore/tree/1.2
2. sudo ./scripts/setup/setup_ubuntu.sh
3. make pre && make
4. ./bin/gserver --start
5. ./bin/ghttp -p 9001

## 在Ubuntu22.04版本下的安装结果

&ensp;&ensp;当看到‘Compilation ends successfully!’之后，则说明已经安装成功，详细截图如下所示，图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/database/gStore/img)

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/04/18/aGEjQDgoVJBifb6.png" alt="gStore1.2-Untuntu22.04安装过程(一)" style="width: 45%;">
    <img src="https://s2.loli.net/2024/04/18/H1roy2au6bTFwPI.png" alt="gStore1.2-Untuntu22.04安装过程(二)" style="width: 45%;">
</div>

## 使用Java语言进行试用

&ensp;&ensp;首先请使用官方提供的工具类[GstoreConnector.java](https://github.com/pkumod/gStore/blob/1.2/api/http/java/src/jgsc/GstoreConnector.java)，在此基础上编写如下单元测试.

&ensp;&ensp;在编写本文的过程中，在使用事务查询的时候遇到了一点问题：事务查询中包含了 insert 和 delete，但是最终数据查询为空，已进行了提交[issues](https://github.com/pkumod/gStore/issues/139).

```java
package cn.net.pap.gstore;

import jgsc.GstoreConnector;
import org.json.JSONObject;
import org.junit.Test;

import static org.junit.jupiter.api.Assertions.assertTrue;

public class GStoreTest {

    public static final String ip = "192.168.1.115";

    public static final Integer port = 9001;

    public static final String httpType = "ghttp";

    public static final String dbName = "kuangbiao";
    public static final String username = "root";
    public static final String password = "123456";

    // 请求格式 json
    public static final String format_json = "json";

    // 请求方式 GET
    public static final String request_type_get = "GET";

    // 请求方式 POST
    public static final String request_type_post = "POST";

    // SPARQL 语句，查询所有
    public static final String SPARQL_SELECT_ALL = "SELECT * WHERE { ?s ?p ?o }";

    // @Test
    public void queryAll() throws Exception {
        GstoreConnector gc = new GstoreConnector(ip, port, httpType, username, password);

        gc.load(dbName, null, request_type_get);

        String res = gc.query(dbName, format_json, SPARQL_SELECT_ALL, request_type_get);
        JSONObject queryJson = new JSONObject(res);
        assertTrue(queryJson.get("StatusCode").toString().equals("0"));

        gc.unload(dbName);

    }

    // @Test
    public void crud() throws Exception {
        String curdDBName = "test";
        // 在服务端执行 cd /home/alexgaoyh;  touch gStore-empty.nt 命令，生成一个空的文件，这样就可以进行 DB 创建
        String dbPathInServer = "/home/alexgaoyh/gStore-empty.nt";

        GstoreConnector gc = new GstoreConnector(ip, port, httpType, username, password);

        // 先删除DB，再创建DB
        String dropDB = gc.drop(curdDBName, false);
        System.out.println(dropDB);
        String buildDB = gc.build(curdDBName, dbPathInServer);
        System.out.println(buildDB);

        // 使用当前 DB
        gc.load(curdDBName, null, request_type_get);

        // 插入三元组数据
        String insertRes = gc.query(curdDBName, format_json, "insert data { " +
                "<人物/#张三> <好友> <人物/#李四>." +
                "<人物/#张三> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <人物>." +
                "<人物/#李四> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <人物>." +
                "<人物/#张三> <性别> \"男\"^^<http://www.w3.org/2001/XMLSchema#String>.\n" +
                "<人物/#张三> <年龄> \"28\"^^<http://www.w3.org/2001/XMLSchema#Int>.\n" +
                "}", request_type_get);
        System.out.println(insertRes);

        String updateRes = gc.query(curdDBName, format_json, "DELETE data {\n" +
                "  <人物/#张三> <性别> \"男\"^^<http://www.w3.org/2001/XMLSchema#String>.\n" +
                "}", request_type_get);
        System.out.println(updateRes);

        String insert2Res = gc.query(curdDBName, format_json, "insert data {\n" +
                "  <人物/#张三> <性别> \"女\"^^<http://www.w3.org/2001/XMLSchema#String>.\n" +
                "}", request_type_get);
        System.out.println(insert2Res);
    }

    // 事务测试
    @Test
    public void transaction() throws Exception {
        String transactionDBName = "transaction";
        // 在服务端执行 cd /home/alexgaoyh;  touch gStore-empty.nt 命令，生成一个空的文件，这样就可以进行 DB 创建
        String dbPathInServer = "/home/alexgaoyh/gStore-empty.nt";
        // 事务隔离级别，串行化
        String isoLevel = "3";
        GstoreConnector gc = new GstoreConnector(ip, port, httpType, username, password);

        // 先删除DB，再创建DB
        String dropDB = gc.drop(transactionDBName, false);
        System.out.println(dropDB);
        String buildDB = gc.build(transactionDBName, dbPathInServer);
        System.out.println(buildDB);

        // 使用当前 DB
        gc.load(transactionDBName, null, request_type_get);

        // 开启事务
        String begin = gc.begin(transactionDBName, isoLevel, request_type_get);
        JSONObject beginJson = new JSONObject(begin);
        System.out.println(begin);
        if(beginJson.get("StatusCode").toString().equals("0")) {
            String tId = beginJson.get("TID").toString();

            try {
                // 插入三元组数据
                String insertRes = gc.tquery(transactionDBName, tId, "insert data { " +
                        "<人物/#张三> <好友> <人物/#李四>." +
                        "<人物/#张三> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <人物>." +
                        "<人物/#李四> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <人物>." +
                        "<人物/#张三> <性别> \"男\"^^<http://www.w3.org/2001/XMLSchema#String>.\n" +
                        "<人物/#张三> <年龄> \"28\"^^<http://www.w3.org/2001/XMLSchema#Int>.\n" +
                        "}", request_type_get);
                System.out.println(insertRes);

                String insert2Res = gc.tquery(transactionDBName, tId, "insert data {\n" +
                        "  <人物/#张三> <性别> \"女\"^^<http://www.w3.org/2001/XMLSchema#String>.\n" +
                        "}", request_type_get);
                System.out.println(insert2Res);

                String updateRes = gc.tquery(transactionDBName, tId, "delete data {\n" +
                        "<人物/#张三> <性别> \"男\"^^<http://www.w3.org/2001/XMLSchema#String>." +
                        "}", request_type_get);
                System.out.println(updateRes);

                String commit = gc.commit(transactionDBName, tId);
                System.out.println(commit);

            } catch (Exception e) {
                String rollback = gc.rollback(transactionDBName, tId);
                System.out.println(rollback);
            }
        }

        String checkpoint = gc.checkpoint(transactionDBName);
        System.out.println(checkpoint);

        String res = gc.query(transactionDBName, format_json, SPARQL_SELECT_ALL, request_type_get);
        System.out.println(res);

    }
}

```

## gWorkbench图数据库管理端

&ensp;&ensp;作者提交了gWorkbench的试用申请，并将服务部署至Tomcat容器之后，使用官方提供的'狂飙'的三元组数据，相关展示如下所示，图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/database/gStore/img)

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/04/18/ZU19wpKBYSCciHE.jpg" alt="gWorkbench图数据库管理端-狂飙-三元组" style="width: 95%;">
</div>


## 总结

&ensp;&ensp;目前来看，gStore在非企业版使用过程中相比Neo4j仍缺少比如路径查询等方法的支持，但是基本的数据查询和展示能满足绝大部分需求。

## 参考

1. https://github.com/pkumod/gStore
2. http://pap-docs.pap.net.cn/
3. https://gitee.com/alexgaoyh








