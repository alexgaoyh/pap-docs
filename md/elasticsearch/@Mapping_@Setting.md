# ElasticSearch @org.springframework.data.elasticsearch.annotations.Field 注解

    在使用 Spring Boot ElasticSearch 的过程中，发现单独的 @Field 注解存在不足，比如没有办法自定义分词器，需要额外的进行配置，
        极端情况下有可能同一个索引需要在多个地方维护，维护起来较为麻烦，所以考虑增加如下的注解。 
            @Mapping(mappingPath = "doc/student-mapping.json")
            @Setting(settingPath = "doc/student-setting.json")

    详见代码。
