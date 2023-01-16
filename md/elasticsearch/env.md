# ElasticSearch 环境配置相关

    根据引入的 spring-boot-starter-data-elasticsearch ，获取 ElasticSearch 的版本，本地使用的是 elasticsearch-7.17.6(此版本建议jdk11)，并且查看匹配的 ik 分词插件，选择相同的版本；

        1、windows下启动命令： elasticsearch.bat
        2、由于可能存在 geoip 数据库的下载，如不使用可以在 elasticsearch.yml 添加： ingest.geoip.downloader.enabled: false
