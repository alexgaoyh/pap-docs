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

- 机器学习/深度学习
  - Paddle
    - [飞浆paddle环境安装](md/other/paddle/paddle-install.md)
    - [ERNIE-Layout 使用测试](md/other/paddle/paddle-ERNIE-Layout.md)
    - [ERNIE-UIE](md/other/paddle/paddle-uie.md)
    - [PaddleGAN-人脸表情迁移](md/other/paddle/PaddleGAN-motion_driving.md)
  - LLM
    - [langchain-ChatGLM](md/other/nlp/langchain-ChatGLM.md)
  - HuggingFace
    - [系列文章(1)-认识Transformers](md/huggingface/install-check.md)
  - 文本提取
    - [基于Apache Tika的文本内容提取](md/tika/tika.md)
- 国产化
  - [银河麒麟v10桌面版FTP适配(FtpClient)](md/localization/kylin/kylin-ftp.md)
- Spring Boot脚手架
  - [PAP4J-BOOT3](md/pap4j_boot3/introduce.md)
- 算法
  - [分类 - 朴素贝叶斯](md/algorithm/algorithm-naivebayes.md)
  - [分类 - KNN](md/algorithm/algorithm-knn.md)
  - [优化 - DE](md/algorithm/algorithm-de.md)
  - [字典数据结构 - Darts-Java-Pos](md/algorithm/algorithm-darts-java-pos.md)
  - [字典数据结构 - FST](md/algorithm/algorithm-fst.md)
  - [图数据结构 - 路径查找](md/algorithm/algorithm-graph-path-search.md)
  - [动态规划-编辑距离-两字符串集合重排序](md/algorithm/algorithm-two-str-list-reorder.md)
  - [动态规划-序列比对-Smith-Waterman](md/algorithm/algorithm-Smith-Waterman.md)
  - [动态规划-序列比对-最长公共子序列](md/algorithm/algorithm-LCS.md)
  - [图像边缘检测-去黑边](md/algorithm/image/remove-black-border.md)
  - [图像边缘检测-自动纠偏](md/algorithm/image/auto-correction.md)
  - [图像处理-锐化](md/algorithm/image/sharpening-prewitt-overlay.md)
  - [图像处理-Java-去噪/高斯模糊/套红](md/algorithm/image/image-denoise-gaussianBlur-red.md)
  - [图像处理-Java-背景色平滑/反色](md/algorithm/image/image-backgroundSmooth-invert.md)
  - [图像处理-Java-指定大小压缩](md/algorithm/image/image-compress-to-target-size.md)
  - [图像处理-Java-TIFF转换JPG](md/algorithm/image/image-tif-convert-jpg.md)
  - [图像处理-Java-字深字浅](md/algorithm/image/image-fontweight-deep-shallow.md)
  - [图像处理-Java-以图搜图](md/algorithm/image/image-search-by-image.md)
  - [字符串操作-逗号分割字符串转树形结构](md/algorithm/algorithm-string-list-to-tree.md)
  - [字符串操作-两个数组之间的重排序](md/algorithm/algorithm-array-resort-by-other.md)
  - [作业调度问题-遗传算法](md/algorithm/genetic-algorithm-job-scheduling.md)
- 系统设计
  - [思考-RBAC中对于权限编码部分的压缩处理](md/design/permission/rethink-rbac-permission-code.md)
  - [思考-RBAC中对于权限编码部分的压缩处理(RoaringBitmap)](md/design/permission/rethink-rbac-permission-code-RoaringBitmap.md)
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
  - Lucene
    - [自定义分词器](md/lucene/combined-analyzer.md)
    - [基于Lucene8进行多值字段分组聚合(多属性字段)](md/lucene/multi-value-field-group-aggregation.md)
    - [基于Lucene8版本的逆向最大匹配分词器(依赖本地词典)](md/lucene/backward-maximum-matching-analyzer.md)
    - [自定义JSON属性值分词器](md/lucene/json-analyzer.md)
  - JPA
    - [动态模型](md/jpa/Hibernate-dynamic-model.md)
- 杂谈
  - [Chrome XSwitch 浏览器请求转发](md/other/chrome-XSwitch-plugin.md)
  - [大型JSON数据切分(Java Jackson)](md/other/big-json-split-in-limited-memory.md)
  - [pdfbox 2.0.28 文字水印移除操作](md/other/pdfbox/remove-text-watermark-pdfbox.md)
  - [从PDF文件中进行表格抽取](md/other/pdfbox/extracte-table-from-file.md)
  - [[国产化低代码平台]基于JPA的简易伪低代码模块](md/other/pap4j-jpa-lowcode.md)
  - [JavaFx项目至安装程序](md/other/sb-project-to-install-program.md)
  - [替代关系型数据库 MAX 聚合函数的思路](md/database/select-max-function-optimize.md)
  - [Excel数据转换为一对多的工具类](md/other/excel/extract-excel-to-multi-object.md)
  - [Excel复杂表头按组按行复制](md/other/excel/excel-copy-template-group.md)
  - [优化-Spring Boot项目服务端接口超时设置](md/other/sb-api-timeout-setting.md)
  - [集合-Java-笛卡尔积、平铺](md/collection/collection-descartes-flat.md)
  - [IDE-idea-可执行JAR项目创建](md/other/idea-exec-jar-no-maven.md)
  - [一种Java语言下的简单重试实现](md/other/simple-retry-impl-in-java.md)
  - [一种Java语言下生成竖版表格文档的方法](md/other/doc/gene-doc-in-direction-tableCell.md)
