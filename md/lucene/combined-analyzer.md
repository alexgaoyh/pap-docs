# [分词]基于Lucene8版本的混合分词器(分词合并)

## 介绍

&ensp;&ensp;近期在研究NLP相关技术，再次感受到中文领域分词算法的重要性，突然想到一年前在项目中使用到的Lucene技术中关于分词器的部分，对其再次进行对比分析，并混合多种现有分词方法，获得更多词项。

## 背景

&ensp;&ensp;目前在Lucene中使用较多的 IK 分词，但遇到古文的话分词效果并不好，会造成检索结果达不到要求。

&ensp;&ensp;在实际使用Lucene的过程中，很难对同一个字段同时使用多种分词算法。

&ensp;&ensp;NGram算法虽然能够获得更多的词项，但是划分的词项太多，造成检索过程中的效率问题。


## 解决方案

&ensp;&ensp;采用自定义分词器的思路，合并多种分词器为一个，从而获得更多词项，后续在整合Lucene的时候，使用此分词器从而得到更好地效果。


## 实现代码

&ensp;&ensp;详见如下测试代码，此代码调用自定义分词器，对输入字符串分别进行 IKAnalyzer 和 WhitespaceAnalyzer 两种分词并合并，如果遇到其他需求，可以合并其他分词器，最终达到理想效果。

&ensp;&ensp;核心代码详见 CustomTokenizer.java 文件。完整代码详见：[pap-lucene-spring-boot-starter](https://gitee.com/alexgaoyh/pap-lucene-spring-boot-starter)

```java
public class CombinedAnalyzerTest {
    public void words() {
        Analyzer analyzer = new CombinedAnalyzer();
        TokenStream tokenStream = null;
        try {
            tokenStream = analyzer.tokenStream("", new StringReader("电视 牛旦 法 水电费 水电费韦尔奇adfmex alexgaoyh 王测试数据"));
            ListAttribute attr = tokenStream.addAttribute(ListAttribute.class);
            tokenStream.reset();
            while (tokenStream.incrementToken()) {
                System.out.println(attr.getValues().toString());
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (tokenStream != null) {
                try {
                    tokenStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

// 如上代码输出(包含了IK分词和空格分词两种结果,并去重)：  [电视, 牛, 旦, 法, 水电费, 韦尔奇, adfmex, alexgaoyh, 王, 测试数据, 牛旦, 水电费韦尔奇adfmex, 王测试数据]
```

```html
public class CustomTokenizer extends Tokenizer {

    private final ListAttribute listAttribute = addAttribute(ListAttribute.class);

    public CustomTokenizer() {
        super();
    }

    @Override
    public final boolean incrementToken() throws IOException {
        clearAttributes();

        if (input.read() != -1) {

            input.reset();
            List<String> ikResultList = ikResult(input);

            input.reset();
            List<String> whitespaceResultList = whitespaceResult(input);

            List<String> mergeList = Stream.of(ikResultList, whitespaceResultList).flatMap(Collection::stream).distinct().collect(Collectors.toList());
            listAttribute.setValues(mergeList);

            return true;
        } else {
            return false;
        }
    }

    private List<String> ikResult(Reader input) throws IOException {
        List<String> tokens = new ArrayList<>();
        Analyzer ikAnalyzer = new IKAnalyzer(true);
        TokenStream ikTokenStream = ikAnalyzer.tokenStream("", input);
        CharTermAttribute ikAttr = ikTokenStream.addAttribute(CharTermAttribute.class);
        ikTokenStream.reset();
        while (ikTokenStream.incrementToken()) {
            tokens.add(ikAttr.toString());
        }
        return tokens;
    }

    private List<String> whitespaceResult(Reader input) throws IOException {
        List<String> tokens = new ArrayList<>();
        Analyzer whitespaceAnalyzer = new WhitespaceAnalyzer();
        TokenStream whitespaceTokenStream = whitespaceAnalyzer.tokenStream("", input);
        CharTermAttribute whitespaceAttr = whitespaceTokenStream.addAttribute(CharTermAttribute.class);
        whitespaceTokenStream.reset();
        while (whitespaceTokenStream.incrementToken()) {
            tokens.add(whitespaceAttr.toString());
        }
        return tokens;
    }

}
```

## 总结

&ensp;&ensp;此代码在Lucene8.6.2版本下通过测试，采用自定义分词器的方式，通过合同多种分词器的方式，达到更广泛的分词结果，从而变相提升检索过程中的覆盖率。

## 参考

1. https://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap-lucene-spring-boot-starter
