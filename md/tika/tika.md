# [文本提取]基于Apache Tika的文本内容提取

## 背景

&ensp;&ensp;近期再次遇到了关于知识库的需求，对照前期使用[langchain-ChatGLM 本地知识库]进行的处理，发现提取文本内容的功能在这个领域中必不可少，故对其进行了研究，通过对比，对Apache Tika 进行知识储备。

## 编码

&ensp;&ensp;使用Spring Maven 与 Apache Tika进行整合，完成对文本内容提取的功能。

&ensp;&ensp;首先在pom.xml引入Tika，其次在resources文件夹下创建tika-config.xml文件，并配置PapTikaConfig.java，最后注入Tika并进行单元测试。

```html
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.tika</groupId>
                <artifactId>tika-bom</artifactId>
                <version>2.9.0</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependency>
        <groupId>org.apache.tika</groupId>
        <artifactId>tika-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.tika</groupId>
        <artifactId>tika-parsers-standard-package</artifactId>
    </dependency>
```

```xml
<properties>
    <encodingDetectors>
        <encodingDetector class="org.apache.tika.parser.html.HtmlEncodingDetector">
            <params>
                <param name="markLimit" type="int">64000</param>
            </params>
        </encodingDetector>
        <encodingDetector class="org.apache.tika.parser.txt.UniversalEncodingDetector">
            <params>
                <param name="markLimit" type="int">64001</param>
            </params>
        </encodingDetector>
        <encodingDetector class="org.apache.tika.parser.txt.Icu4jEncodingDetector">
            <params>
                <param name="markLimit" type="int">64002</param>
            </params>
        </encodingDetector>
    </encodingDetectors>
</properties>

```

```java
@Configuration
public class PapTikaConfig {
    @Autowired
    private ResourceLoader resourceLoader;

    @Bean
    public Tika tika() throws TikaException, IOException, SAXException {

        Resource resource = resourceLoader.getResource("classpath:tika-config.xml");
        InputStream inputStream = resource.getInputStream();

        TikaConfig config = new TikaConfig(inputStream);
        Detector detector = config.getDetector();
        Parser autoDetectParser = new AutoDetectParser(config);

        return new Tika(detector, autoDetectParser);
    }
}
```

```html
    @Autowired
    private Tika tika;

    @Test
    void tika() throws Exception {
        String[] filePathArray = new String[]{"pap.pptx", "pap.txt", "pap.xlsx", "pap.docx", "pap.pdf"};
        for(String filPath : filePathArray) {
            String s = tika.parseToString(new File(filPath));
        }
    }
```

## 总结

&ensp;&ensp;在NLP领域，存在对大量的文本文件进行内容提取的场景，此组件能满足绝大部分场景。

&ensp;&ensp;同样可以进行元数据的读取，本文不对此进行特别介绍，可自行查找资料。

## 参考

1. https://gitee.com/alexgaoyh/pap4j-tika
2. https://tika.apache.org/
