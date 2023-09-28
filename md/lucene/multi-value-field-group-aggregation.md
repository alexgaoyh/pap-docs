# [分组聚合]基于Lucene8进行多值字段分组聚合(多属性字段)

## 介绍

&ensp;&ensp;在使用搜索引擎的过程中，经常会出现针对属性进行分组聚合的场景，单属性分组聚合很简单，如何对多属性字段进行分组聚合是本文的重点。

## 背景

&ensp;&ensp;在真实世界中，任意一个物品的属性可能会有多个，形如学生有多个兴趣爱好，如何根据兴趣爱好进行分组？如果高效的解决类似的需求尤为重要。


## 解决方案

&ensp;&ensp;1、仍然按照单字段单属性分组聚合的方式，对分组聚合的结果进行循环处理，从而获得期望的结果。

&ensp;&ensp;2、引入 lucene-facet 对多属性字段进行分组统计。本文主要介绍此方法。

## 实现代码

&ensp;&ensp;详见如下测试代码

&ensp;&ensp;其中针对 Category 字段分组查询之后得到的结果为：gao : 2 ; alex,gao : 1 ; alex,pap : 1

&ensp;&ensp;其中针对 MCategory 字段分组查询之后得到的结果为：gao : 3 ; alex : 2 ; pap : 1


```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {com.pap.lucene.LuceneAutoConfiguration.class})
@TestPropertySource("classpath:application.properties")
@FixMethodOrder(MethodSorters.JVM)
public class LuceneAutoConfigurationTest {

    @Autowired
    private SearcherManager searcherManager;
    @Autowired
    private IndexWriter indexWriter;
    @Autowired
    private DirectoryTaxonomyWriter taxoWriter;

    @Test
    public void init() throws Exception {
        FacetsConfig facetsConfigInit = new FacetsConfig();
        facetsConfigInit.setMultiValued("MCategory", true);

        indexDocument(indexWriter, taxoWriter, facetsConfigInit, "pap.net.cn 1", "pap.net.cn of content 1", "alex,gao", "Tag A");
        indexDocument(indexWriter, taxoWriter, facetsConfigInit, "pap.net.cn 2", "pap.net.cn of content 2", "alex,pap", "Tag B");
        indexDocument(indexWriter, taxoWriter, facetsConfigInit, "pap.net.cn 3", "pap.net.cn of content 3", "gao", "Tag A");
        indexDocument(indexWriter, taxoWriter, facetsConfigInit, "pap.net.cn 4", "pap.net.cn of content 4", "gao", "Tag B");

        indexWriter.commit();
        taxoWriter.commit();

    }

    @Test
    public void search() throws Exception {
        Query query = new TermQuery(new Term("content", "content"));

        searcherManager.maybeRefresh();
        IndexSearcher indexSearcher = searcherManager.acquire();

        FacetsCollector facetsCollector = new FacetsCollector();
        FacetsCollector.search(indexSearcher, query, 10, facetsCollector);

        FacetsConfig facetsConfigSearch = new FacetsConfig();
        Facets facets = new FastTaxonomyFacetCounts(new DirectoryTaxonomyReader(taxoWriter), facetsConfigSearch, facetsCollector);

        FacetResult category = facets.getTopChildren(10, "Category");
        if(category != null) {
            for(LabelAndValue labelValues : category.labelValues) {
                System.out.println(labelValues.label + " : " + labelValues.value);
            }
        }
        System.out.println("------");
        FacetResult MCategory = facets.getTopChildren(10, "MCategory");
        if(MCategory != null) {
            for(LabelAndValue labelValues : MCategory.labelValues) {
                System.out.println(labelValues.label + " : " + labelValues.value);
            }
        }
    }

    private static void indexDocument(IndexWriter indexWriter, DirectoryTaxonomyWriter taxoWriter, FacetsConfig facetsConfig, String title, String content, String category, String tag) throws IOException {
        Document doc = new Document();
        doc.add(new StringField("title", title, Field.Store.YES));
        doc.add(new TextField("content", content, Field.Store.YES));
        doc.add(new FacetField("Category", category));
        for(String tmp :category.split(",")) {
            doc.add(new FacetField("MCategory", tmp));
        }
        doc.add(new FacetField("Tag", tag));
        indexWriter.addDocument(facetsConfig.build(taxoWriter, doc));
    }
}
```

## 总结

&ensp;&ensp;此代码在Lucene8.6.2版本下通过测试，从而高效的获取到单字段多属性的分组聚合结果。

## 参考

1. https://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap-lucene-spring-boot-starter
