# [分词]基于Lucene8版本的JSON结构分词器(属性值集合)

## 介绍

&ensp;&ensp;在实际场景中，不排除将JSON字符串存储至Database中，并且现有的绝大部分数据库都支持了JSON结构，但是在将关系型数据库中的数据索引至Lucene的时候，针对JSON结构需要进行额外的处理。


## 解决方案

&ensp;&ensp;采用自定义分词器的思路，先将JSON字符串进行解析，获得所有属性值，并将属性值当做结果存储至Lucene中。


## 实现代码

&ensp;&ensp;详见如下测试代码，此代码调用自定义的JSON分词器。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {com.pap.lucene.LuceneAutoConfiguration.class})
@TestPropertySource("classpath:application.properties")
@FixMethodOrder(MethodSorters.JVM)
public class JSONTest {

    @Autowired
    private SearcherManager searcherManager;
    @Autowired
    private IndexWriter indexWriter;

    @Test
    public void init() throws Exception {
        indexWriter.deleteAll();
        indexDocument(indexWriter, "{\"name\":\"alexgaoyh\",\"sex\":\"male\",\"city\":\"Xu Chang\",\"phones\":[\"alexgaoyh@sina.com\",\"pap.net.cn\"],\"ext\":{\"skill\":\"programmer\",\"zip\":\"461000\",\"remark\":{\"language\":[\"chinese\",\"english\"],\"main-development\":\"java\"}}}");
        indexWriter.commit();
        // 分词结果:  ['alexgaoyh','male','XuChang','alexgaoyh@sina.com','pap.net.cn','programmer','461000','chinese','english','java']
    }

    @Test
    public void searchInOrder() throws Exception {
        TermQuery termQuery = new TermQuery(new Term("json", "alexgaoyh"));
        searcherManager.maybeRefresh();
        IndexSearcher indexSearcher = searcherManager.acquire();
        TopDocs search = indexSearcher.search(termQuery, 10);
    }

    private static void indexDocument(IndexWriter indexWriter, String json) throws IOException {
        Document doc = new Document();
        doc.add(new TextField("json", json, Field.Store.YES));
        doc.add(new SortedDocValuesField("json", new BytesRef(json)));
        indexWriter.addDocument(doc);
    }
}

```

```html
public class JSONAnalyzer extends Analyzer {

    @Override
    protected TokenStreamComponents createComponents(String s) {
        Tokenizer tokenizer = new JSONTokenizer();
        return new TokenStreamComponents(tokenizer);
    }

}

public final class JSONTokenizer extends Tokenizer {

    private final CharTermAttribute charTermAttribute = addAttribute(CharTermAttribute.class);
    private final OffsetAttribute offsetAttribute = addAttribute(OffsetAttribute.class);

    private Iterator<Map<String, Object>> termIterator;

    public JSONTokenizer() {
        super();
    }

    @Override
    public boolean incrementToken() throws IOException {
        clearAttributes();
        if (termIterator == null) {
            int intValueOfChar;
            String text = "";
            while ((intValueOfChar = input.read()) != -1) {
                text += (char) intValueOfChar;
            }
            List<Map<String, Object>> terms = JsonValueExtractor.segment(text);
            termIterator = terms.iterator();
        }
        if (termIterator.hasNext()) {
            Map<String, Object> termMap = termIterator.next();
            charTermAttribute.append(termMap.get("word").toString());
            charTermAttribute.setLength(termMap.get("word").toString().length());
            offsetAttribute.setOffset(correctOffset(Integer.parseInt(termMap.get("offset").toString())), correctOffset(Integer.parseInt(termMap.get("offset").toString()) + termMap.get("word").toString().length()));
            return true;
        } else {
            termIterator = null;
            return false;
        }
    }
}

public class JsonValueExtractor {

    public static List<Map<String, Object>> segment(String jsonString) {
        List<Map<String, Object>> segmentMapList = new ArrayList<>();
        List<String> values = extractValuesUsingJava(jsonString);
        if (values != null && values.size() > 0) {
            for (String term : values) {
                Map<String, Object> segmentMap = new HashMap<>();
                segmentMap.put("word", term);
                segmentMap.put("offset", jsonString.indexOf(term));
                segmentMapList.add(segmentMap);
            }
        }
        return segmentMapList;
    }

    public static List<String> extractValuesUsingJava(String jsonString) {
        List<String> values = new ArrayList<>();
        try (JsonReader jsonReader = Json.createReader(new StringReader(jsonString))) {
            JsonValue jsonValue = jsonReader.read();
            traverseJsonValueUsingJava(jsonValue, values);
        }
        return values;
    }

    private static void traverseJsonValueUsingJava(JsonValue jsonValue, List<String> values) {
        switch (jsonValue.getValueType()) {
            case OBJECT:
                JsonObject jsonObject = (JsonObject) jsonValue;
                for (Map.Entry<String, JsonValue> entry : jsonObject.entrySet()) {
                    traverseJsonValueUsingJava(entry.getValue(), values);
                }
                break;
            case ARRAY:
                JsonArray jsonArray = (JsonArray) jsonValue;
                for (JsonValue element : jsonArray) {
                    traverseJsonValueUsingJava(element, values);
                }
                break;
            default:
                values.add(jsonValue.toString().substring(1, jsonValue.toString().length() - 1));
                break;
        }
    }
}
```


## 总结

&ensp;&ensp;此代码在Lucene8.6.2版本下通过测试，采用自定义JSON分词器的方式，对JSON结构的数据提升支持力度。

## 参考

1. https://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap-lucene-spring-boot-starter
