# ElasticSearch IK

    下载 ik 分词器，在 plugins 文件夹下创建目录 ik ， 将插件拷贝进来：
    
        plugins\ik
            config
            commons-codec-1.9.jar
            commons-logging-1.2.jar
            elasticsearch-analysis-ik-7.17.6.jar
            httpclient-4.5.2.jar
            httpcore-4.4.4.jar
            plugin-descriptor.properties
            plugin-security.policy
        
        测试分词结果:
        
            curl --location --request POST 'http://localhost:9200/_analyze' \
                --header 'Content-Type: application/json' \
                --data-raw '{
                    "analyzer": "ik_max_word",
                    "text": "我是中文，我要看看分词的效果"
                }'
    
        分词热加载
            
            plugins\ik\config\IKAnalyzer.cfg.xml 文件可以配置 remote_ext_dict参数，可以对其进行控制，
    
                1、JAVA测试代码如下：
    
                    @RequestMapping(value = "/hot", method = {RequestMethod.GET,RequestMethod.HEAD}, produces="text/html;charset=UTF-8")
                    public String getHotWordByOracle(HttpServletResponse response, String filePath) throws IOException {
                        Map<String, Object> map = readInfoFromFilePath(filePath);
                
                        response.setHeader("Last-Modified",map.get("Last-Modified").toString());
                        response.setHeader("ETag",map.get("md5").toString());
                
                        return map.get("content").toString();
                    }
                
                    /**
                     * 文件信息，包括文件上次修改值，文件md5值， 文件内容
                     *
                     * @param filePath 文件绝对路径
                     * @return Last-Modified 上次修改时间 ；  md5 文件md5值 ； content 文件内容
                     * @throws IOException
                     */
                    public static Map<String, Object> readInfoFromFilePath(String filePath) throws IOException {
                
                        Map<String, Object> infoMap = new HashMap<>(3);
                        StringBuilder words = new StringBuilder();
                
                        File file = new File(filePath);
                        long fileLastModified = file.lastModified();
                        String fileMd5 = getMD5(file);
                
                        List<String> contentList = Files.readAllLines(Paths.get(filePath));
                
                        if(contentList != null && contentList.size() > 0) {
                            for(String str : contentList) {
                                words.append(str);
                                words.append("\n");
                            }
                        }
                
                        infoMap.put("Last-Modified", fileLastModified);
                        infoMap.put("md5", fileMd5);
                        infoMap.put("content", words);
                
                        return infoMap;
                    }
                
                    /**
                     * 文件 MD5
                     *
                     * @param file
                     * @return
                     */
                    public static String getMD5(File file) {
                        FileInputStream fileInputStream = null;
                        try {
                            MessageDigest MD5 = MessageDigest.getInstance("MD5");
                            fileInputStream = new FileInputStream(file);
                            byte[] buffer = new byte[8192];
                            int length;
                            while ((length = fileInputStream.read(buffer)) != -1) {
                                MD5.update(buffer, 0, length);
                            }
                            return new String(Hex.encodeHex(MD5.digest()));
                        } catch (Exception e) {
                            e.printStackTrace();
                            return null;
                        } finally {
                            try {
                                if (fileInputStream != null) {
                                    fileInputStream.close();
                                }
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                    }
    
                2、Nginx 配置（当前使用 nginx-1.20.2），进行如下代码段落配置，测试的话查看响应头的 Last-Modified ETag
    
                    server {
                        listen       81;
                        server_name  localhost;
                        
                        charset utf-8;
                        
                        location / {
                            charset utf-8;
                            root   xxxx;
                        }
                    }
                
                3、修改 elasticsearch-analysis-ik 源码，增加从 database 加载数据的方法：
                    个人不建议采取这种方式，原因在于感觉会增加 ES 的负担，毕竟增加了数据库相关的逻辑；                        
    
                    https://gitee.com/alexgaoyh/elasticsearch-analysis-ik/commit/19a07dbc31a1f8f82b4f4da8e82bcd10e098b3ca
    
                    1、从 https://github.com/medcl/elasticsearch-analysis-ik.git 同步代码，并根据当前 es 的版本，切换Tag 到 v7.17.6;
                    2、更改部分详见 如上 commit 的提交代码；
                    3、执行 package 进行打包，之前就可以替换，并放置到 es/plugins/ik 目录下；
                    
                        如果出现 java.security.AccessControlException: access denied (?java.lang.RuntimePermission? setContextClassLoader?) 的错误
                        进入java 安装目录，作者环境是 D:\Java\jdk1.8.0_281\jre\lib\security ，修改 java.policy 文件，最后增加 permission java.security.AllPermission; 
