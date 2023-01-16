# ElasticSearch Ingest Attachment Processor Plugin

    https://www.elastic.co/guide/en/elasticsearch/plugins/7.17/ingest.html
    https://www.elastic.co/guide/en/elasticsearch/plugins/7.17/ingest-attachment.html
    
    https://artifacts.elastic.co/downloads/elasticsearch-plugins/ingest-attachment/ingest-attachment-7.17.6.zip


        1、首先创建pipeline
            curl --location --request PUT 'http://localhost:9200/_ingest/pipeline/attachment' \
                --header 'Content-Type: application/json' \
                --data-raw '{
                    "description": "Extract single attachment information",
                    "processors": [
                        {
                            "attachment": {
                                "field": "fileBase64",
                                "indexed_chars": -1,
                                "ignore_missing": true
                            }
                        },
                        {
                            "remove": {
                                "field": "fileBase64"
                            }
                        }
                    ]
                }'
        
        2、使用 ElasticsearchClient 进行测试，之后可以在 elasticsearch head 上查看对应的文档

                @Test
                void ingestPipeline() throws IOException {
                    Student student = new Student("ingestPipeline", "群众", 12, new Date(), "我是中文，我要看看分词的效果");
            
                    // file to base64
                    String fileBase64 = StringUtils.fileToBase64("C:\\Users\\86181\\Desktop\\new 2.txt");
                    student.setFileBase64(fileBase64);
            
                    CreateResponse response = elasticsearchClient.create(e -> e.index("student").pipeline("attachment").document(student).id("ingestPipeline"));
                }
