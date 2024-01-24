# [分词]基于Lucene8版本的逆向最大匹配分词器(依赖本地词典)

## 介绍

&ensp;&ensp;在搜索相关业务场景中，强依赖中文分词的结果，本文是[基于Lucene8版本的混合分词器(分词合并)](https://pap-docs.pap.net.cn/#/md/lucene/combined-analyzer)的补充，可以将自定义分词的结果进行添加。


## 实现代码

```java
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenFilter;
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.core.LowerCaseFilter;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.apache.lucene.analysis.tokenattributes.OffsetAttribute;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.HashSet;
import java.util.Set;

public class ReverseMaximumMatchingAnalyzer extends Analyzer {
    private Set<String> dictionary;

    public ReverseMaximumMatchingAnalyzer(Path dictionaryPath) {
        this.dictionary = loadDictionary(dictionaryPath);
    }

    private Set<String> loadDictionary(Path dictionaryPath) {
        Set<String> dictionary = new HashSet<>();
        try (InputStream inputStream = Files.newInputStream(dictionaryPath);
             InputStreamReader streamReader = new InputStreamReader(inputStream, StandardCharsets.UTF_8);
             BufferedReader reader = new BufferedReader(streamReader)) {
            String line;
            while ((line = reader.readLine()) != null) {
                dictionary.add(line.trim());
            }
        } catch (Exception e) {
        }
        return dictionary;
    }

    @Override
    protected TokenStreamComponents createComponents(String fieldName) {
        Tokenizer tokenizer = new ReverseMaximumMatchingTokenizer(dictionary);
        TokenFilter filter = new LowerCaseFilter(tokenizer);
        return new TokenStreamComponents(tokenizer, filter);
    }

    private static class ReverseMaximumMatchingTokenizer extends Tokenizer {
        private char[] buffer = new char[256];
        private int length = 0;
        private int offset = 0;

        private CharTermAttribute termAttribute = addAttribute(CharTermAttribute.class);
        private OffsetAttribute offsetAttribute = addAttribute(OffsetAttribute.class);

        private Set<String> dictionary;

        public ReverseMaximumMatchingTokenizer(Set<String> dictionary) {
            this.dictionary = dictionary;
        }

        @Override
        public boolean incrementToken() throws IOException {
            while (true) {
                if (offset < length) {
                    clearAttributes();
                    int start = offset;
                    while (offset < length) {
                        offset++;
                        int end = offset;
                        while (end >= start) {
                            String token = new String(buffer, start, end - start);
                            if (dictionary.contains(token)) {
                                termAttribute.copyBuffer(buffer, start, end - start);
                                offsetAttribute.setOffset(correctOffset(start), correctOffset(end));
                                return true;
                            }
                            end--;
                        }
                    }
                } else if (input.read(buffer) != -1) {
                    length = buffer.length;
                    offset = 0;
                } else {
                    return false;
                }
            }
        }

        @Override
        public void reset() throws IOException {
            super.reset();
            offset = length = 0;
        }

        @Override
        public void end() throws IOException {
            super.end();
            offset = length;
        }
    }
}
```


## 总结

&ensp;&ensp;此代码在Lucene8.6.2版本下通过测试，对中文分词补充基于逆向最大匹配的分词结果，从而提升质量。

## 参考

1. https://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap-lucene-spring-boot-starter
