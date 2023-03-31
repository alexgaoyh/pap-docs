# ElasticSearch 拼音搜索自定义扩展插件(长拼音序列)。

&ensp;&ensp;当前的中文搜索引擎都支持拼音搜索，但是在使用过程中会出现各种各样的场景，当前的自定义插件，用来处理在拼音搜索过程中如果遇到长拼音序列的情况，应该如何进行处理。

## 概述
&ensp;&ensp;在进行拼音搜索的过程中，会有各种各样的输入方式，这些连续的拼音输入会形成一个长拼音序列，想进行正确的搜索，首要面对的一个问题是如何对一个长拼音序列进行切分。

## 拼音序列输入
&ensp;&ensp;长拼音序列的划分会有很多种场景，这里简单描述几种常见的场景：

1. 普通情况。
    1. 完整的、无错的、没有歧义的拼音序列，可以进行直接处理。
2. 拼音缩写。
    1. 完全简写 ：只输入拼音的首字母，较为常见。
    2. 先整后简 ：前一个字是完整拼音，后一个字是拼音缩写。
    3. 先简后整 ：比较少见，一般出现在想要使用「完全简写」的方式输入，但是发现匹配不到，因此再输入后一个字的完整拼音。

&ensp;&ensp;在ElasticSearch中，有现有的[拼音插件](https://github.com/medcl/elasticsearch-analysis-pinyin)，支持不同版本的ElasticSearch。
经过分析，上述插件在遇到‘先整后简’、‘先简后整’的长序列，是无法搜索出来正常结果的，经过分析，如果找到一种方法，能够将长拼音序列进行切分，其实就可以满足结果。
比如将 'wszgr'切分为 ‘w’、‘s’、‘z’、‘g’、‘r’，使用这个结果进行搜索，就把原问题进行了简化，仍使用上述的[拼音插件](https://github.com/medcl/elasticsearch-analysis-pinyin)就可以解决这个问题。

&ensp;&ensp;综上所述，为了解决长拼音序列的搜索问题，就需要解决两个细分问题：1、拼音切分；2、Elasticsearch自定义分词插件。

## 拼音切分

&ensp;&ensp;先看如下的[分词结果](https://gitee.com/alexgaoyh/elasticsearch-analysis-pinyincut/blob/master/src/main/java/com/pap/pinyincut/core/algorithm/core/PinyinSplitter.java)，使用的思路是类似在小学阶段学习的拼音表进行的实现，划分声母和韵母，然后根据规则进行切分，拼音序列的切分结果如下。

```html
woszgr -> [wo, s, z, g, r]
wszhonggren -> [w, s, zhong, g, ren]
qianb -> [qian, b]
qbi -> [q, bi]
```

&ensp;&ensp;中文拼音表中包含23个声母和24个韵母，进行组合可以得到大量的拼音序列，可以很容易得到所有的拼音序列，如下所示部分的拼音。

```html
声母： b, p, m, f, d, t, n, l, g, k, h, j, q, x, zh, ch, sh, r, z, c, s, y, w
韵母： a, o, e, i, u, ü, ai, ei, ui, ao, ou, iu, ie, üe, er, an, en, in, un, ün, ang, eng, ing, ong
所有的拼音序列： ba, bo, bei, bu, bai, bei, bao, bi, bo, bu, bai, bei, bao, ban, ben, bin, bun, bang, beng, bing ……………………
```

## ElasticSearch 自定义分词插件

&ensp;&ensp;这一部分，互联网上存在着大量的介绍，本章节不对其进行详细介绍，只把核心代码展示出来，详见[源代码](https://gitee.com/alexgaoyh/elasticsearch-analysis-pinyincut)

```java
public class PinyinCutAnalysisPlugin extends Plugin implements AnalysisPlugin {

    public static String PLUGIN_NAME = "analysis-pycut";

    @Override
    public Map<String, AnalysisModule.AnalysisProvider<TokenizerFactory>> getTokenizers() {
        Map<String, AnalysisModule.AnalysisProvider<TokenizerFactory>> extra = new HashMap<>();
        extra.put("pycut_tokenizer", PinyinCutTokenizerFactory::getTokenizerFactory);
        return extra;
    }

    @Override
    public Map<String, AnalysisModule.AnalysisProvider<AnalyzerProvider<? extends Analyzer>>> getAnalyzers() {
        Map<String, AnalysisModule.AnalysisProvider<AnalyzerProvider<? extends Analyzer>>> extra = new HashMap<>();
        extra.put("pycut_analyzer", PinyinCutAnalyzerProvider::getAnalyzerProvider);
        return extra;
    }
}
```

&ensp;&ensp;详情可见[发行版](https://gitee.com/alexgaoyh/elasticsearch-analysis-pinyincut/releases)地址，如果使用的是其他版本的ElasticSearch，则clone源代码，更改pom文件中对应的ElasticSearch版本，重新进行 maven clean package 命令。

## 搜索示例
&ensp;&ensp;第一段代码是settings配置，第二段代码是mapping配置。
&ensp;&ensp;使用如上配置的话，搜索 ’zguo‘、’zhongg‘、’zg‘ ，是都能匹配到 ’中国‘ 这个结果的。
```json
{
  "analysis": {
    "analyzer": {
      "pinyin_own_analyzer" : {
        "tokenizer" : "pinyin_own_letter_tokenizer"
      }
    },
    "tokenizer": {
      "pinyin_own_letter_tokenizer": {
        "type" : "pinyin",
		"keep_first_letter": false,
        "keep_separate_first_letter" : true,
        "keep_full_pinyin" : true,
		"keep_joined_full_pinyin": false,
        "lowercase" : true,
        "remove_duplicated_term" : true
      }
    }
  }
}
```
```json
{
   "pyOwn": {
      "type": "text",
      "search_analyzer": "pycut_analyzer",
      "analyzer": "pinyin_own_analyzer"
   }
}
```

## 疑问

&ensp;&ensp;在使用[拼音插件](https://github.com/medcl/elasticsearch-analysis-pinyin)的时候，如果遇到 ’zh‘、’ch‘、’sh‘ 这种声母，拼音插件返回的是’z‘、’c‘、’s‘，跟小学阶段学习到的结果不太一样，需要注意。

&ensp;&ensp;本文所写的[拼音扩展插件](https://gitee.com/alexgaoyh/elasticsearch-analysis-pinyincut)也由于上述的情况做了特殊的处理，即便拼音切分出来了 ’zh‘、’ch‘、’sh‘，分词的结果会更改为’z‘、’c‘、’s‘。

## 参考

1. https://github.com/medcl/elasticsearch-analysis-pinyin
2. https://github.com/OrangeX4/simple-pinyin
3. https://gitee.com/alexgaoyh/elasticsearch-analysis-pinyincut
