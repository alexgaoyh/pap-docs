# PAP

- 开源项目
  - [alexgaoyh/alexgaoyh](https://gitee.com/alexgaoyh/alexgaoyh)
    - 整合shiro，ueditor，freemarker模板,ehcache缓存逻辑，增加查询缓存，二级缓存。
    - springmvc4.x hibernate4.x mysql5.x shiro ehcache ueditor freemarker redis2.6 maven
    - [![star](https://gitee.com/alexgaoyh/alexgaoyh/badge/star.svg?theme=dark)](https://gitee.com/alexgaoyh/alexgaoyh/stargazers)[![fork](https://gitee.com/alexgaoyh/alexgaoyh/badge/fork.svg?theme=dark)](https://gitee.com/alexgaoyh/alexgaoyh/members)

  - [alexgaoyh/MutiModule-parent](https://gitee.com/alexgaoyh/MutiModule-parent)
    - Dubbo
    - Maven多模块
      - 富文本编辑器： kindeditor/udeitor
      - 组件： lucene/logback/Hadoop/Captcha/Echarts/RPC
      - 模块： CMS/Upload/CMS/CitySelect/SSO
    - [![star](https://gitee.com/alexgaoyh/MutiModule-parent/badge/star.svg?theme=dark)](https://gitee.com/alexgaoyh/MutiModule-parent/stargazers)[![fork](https://gitee.com/alexgaoyh/MutiModule-parent/badge/fork.svg?theme=dark)](https://gitee.com/alexgaoyh/MutiModule-parent/members)

  - [alexgaoyh/pap-all-project](https://gitee.com/alexgaoyh/pap-all-project)
    - 自定义starter
      - pap-logback-operdb-spring-boot-starter  Interceptor 注解，异步持久化到日志数据库，解决操作日志问题
      - pap-sequence-starter 流水号生成器，形如 年月日自增流水号(可增加前缀与后缀)
      - pap-bean-spring-boot-starter 自定义业务Bean，形如 IDWorker序号生成器/当前登录用户处理Bean
    - LogBack
      -  pap-logback-ext 在持久化数据库的过程中，对logback-core扩展，使其可以最多保存32个参数。 对时间格式进行可视化处理。
      -  pap-logback-oper 多数据源下日志查询(为防止日志持久化到多个不同数据库)，这里需要动态切换数据源。
    - 基础组件
      -  pap-calculate 自定义数学公式计算器，以实现绝大部分Excel中的公式。
      -  pap-code-generator 代码生成器，支持Mybatis自定义扩展插件、FreeMarker 自定义模板。
      -  pap-upload 同一文件处理，支持 ElementUI 。
      -  pap-obj 基础POJO类、含部分工具类
      -  pap-base 基础Base通用方法与额外工具类
      -  pap-activitiy 工作流处理逻辑，以实现 发布模型，发布任务，领取任务，审核任务，任务流程图，与RBAC用户体系打通。
      -  pap-rabbitmq 消息中间件工具类，其中包含 分布式事务过程中针对消息中间件的支持，增加 事务协调者(守护进程) 的概念，保证分布式事务的最终一致性。
      -  pap-spring-boot-spi-demo Saas 平台在设计的过程中，会出现不同租户的业务需求不一致的情况，在这种情况下，则可以考虑使用 SPI 来支持定制化的服务。
      -  pap-spring-boot-admin 管理和监控Spring Boot 应用程序的开源软件，增加Eureka 的支持，并且处理项目上下文的情况。
    - 业务组件
      -  pap-datas 省市区、数据字典功能 。
      -  pap-rbac RBAC用户权限模型，其中因为前后端分离，对RBAC进行部分修改，采用 权限码 进行前后端的权限约定，对各类资源都采用如上的 权限编码 概念进行权限控制。
      -  pap-msg 系统消息、站内信。
      -  pap-customer 客户中心。
      -  pap-item 电商环节，商品中心(其中包含SKU 的处理逻辑，详见子模块的ReadMe 文件)。
      -  pap-product 产品中心，对自定义公式的汇总，可以通过自定义复杂的公式，进行四则运算，可以满足形如 薪酬计算 等需要公式介入的功能。
    - [![star](https://gitee.com/alexgaoyh/pap-all-project/badge/star.svg?theme=dark)](https://gitee.com/alexgaoyh/pap-all-project/stargazers)[![fork](https://gitee.com/alexgaoyh/pap-all-project/badge/fork.svg?theme=dark)](https://gitee.com/alexgaoyh/pap-all-project/members)

- 中间件
  - 缓存
    - [Redis](md/cache/cache-redis.md)
    - [Caffeine](md/cache/cache-caffeine.md)
    - [Caffeine + Redis](md/cache/cache-caffeine-redis.md)
  - Elasticsearch
    - [环境配置](md/elasticsearch/env.md)
    - [IK 分词器](md/elasticsearch/ik.md)
    - [Ingest Attachment Processor Plugin](md/elasticsearch/ingest-attachment.md)
    - [Synonym 同义词](md/elasticsearch/synonym.md)
    - [@Mapping @Setting](md/elasticsearch/@Mapping_@Setting.md)
    - [分组](md/elasticsearch/group.md)
    - [constant_score](md/elasticsearch/constant_score.md)
    - [DIS_MAX](md/elasticsearch/dis_max.md)
    - [copy_to](md/elasticsearch/copy_to.md)
    - [match_phrase.slop](md/elasticsearch/match_phrase.slop.md)
    - [function_score](md/elasticsearch/function_score.md)
    - [boosting](md/elasticsearch/boosting.md)
    - [锁 ](md/elasticsearch/lock.md)
    - [Nested Object](md/elasticsearch/nested.md)
    - [父子文档](md/elasticsearch/parent_child.md)
    - [completion suggest](md/elasticsearch/completion_suggest.md)
    - [dynamic_templates](md/elasticsearch/dynamic_templates.md)
    - [实际场景-具体应用](md/elasticsearch/using_case.md)
    - [_bulk 使用与实战](md/elasticsearch/bulk.md)
    - [Enrich-Processor(Mysql.Join)](md/elasticsearch/Enrich-Processor.md)
    - [长拼音序列切分搜索自定义分词插件](md/elasticsearch/pinyin-cutting.md)
    - [根据词频排序的自定义相似度插件](md/elasticsearch/similarity-tf.md)
    - [逗号分割/集合对象的分组聚合](md/elasticsearch/group-comma-nested.md)
    - [高级检索-按照搜索条件顺序执行](md/elasticsearch/high-query-by-condition-order.md)
- 算法
  - [分类 - 朴素贝叶斯](md/algorithm/algorithm-naivebayes.md)
  - [分类 - KNN](md/algorithm/algorithm-knn.md)
  - [优化 - DE](md/algorithm/algorithm-de.md)
  - [字典数据结构 - Darts-Java-Pos](md/algorithm/algorithm-darts-java-pos.md)
  - [字典数据结构 - FST](md/algorithm/algorithm-fst.md)
  - [图数据结构 - 路径查找](md/algorithm/algorithm-graph-path-search.md)
- 杂谈
  - [Chrome XSwitch 浏览器请求转发](md/other/chrome-XSwitch-plugin.md)
