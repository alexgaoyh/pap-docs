# 字典数据结构 FST(Finite State Transducer)

## 概述
&ensp;&ensp;有限状态自动机（Finite State Transducer，FST）是一种常见的字典数据结构，常用于NLP中。它可以表示一组字符串集合，并提供一种有效的方法来在这些字符串上执行查询操作。

&ensp;&ensp;FST 可以用于多种不同的任务，包括词形变化、拼写纠正、文本匹配和词义消歧等。

## 对比与实现

&ensp;&ensp;由于中文文本自身的特殊性，如何对文本进行正确的分词尤为重要，就牵扯到如何构建字典，常用的字典数据结构如下：

| 数据结构              | 优缺点                                                                        | 相关代码                                                                                                                                                                           |
|-------------------|----------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ArrayList……	      | 二分法查找，不平衡                                                                  | list.add("")                                                                                                                                                                   |
| Map……             | 内存消耗大                                                                      | map.put("","")                                                                                                                                                                 |
| Trie              | 中文领域过于消耗内存                                                                 | [TrieChineseTokenizer.java](https://gitee.com/alexgaoyh/pap-base/blob/14564f3ef0d58f483b0fb5c843fa594bb961f6a8/src/main/java/com/pap/base/util/trie/TrieChineseTokenizer.java) |
| Double Array Trie | 相比于Trie，更适合中文，缺点是无法动态增删                                                    | [darts-java扩展](https://gitee.com/alexgaoyh/darts-java/tree/feature-pos-ext/)                                                                                                   |
| AhoCorasickDoubleArrayTrie | [存储大辞典时溢出](https://github.com/hankcs/AhoCorasickDoubleArrayTrie/issues/38) | [AhoCorasickDoubleArrayTrie](https://github.com/hankcs/AhoCorasickDoubleArrayTrie)                                             |
| Finite State Transducers (FST) | Lucene中大量应用，***本文重点说明***                                                         | [FST.java](https://gitee.com/alexgaoyh/pap-base/tree/051988dd354ffc3f62f8d9ee70710ad3802cca7f/src/main/java/com/pap/base/util/fst)                                             |


1. 针对ArrayList、Map这两个基本数据结构，不做过多介绍，直接跳过。
2. 针对Trie这个数据结构，他更适合于英文，并不适合应用于中文环境。
   1. 中文文本中的字符集非常大，超过了 ASCII 字符集。Trie 数据结构在处理大字符集时，需要存储大量的节点，这会导致空间消耗变得非常大。
   2. 中文文本中的词汇组合非常灵活。中文中的词汇不像英文那样明确定义，并且同一个词可以有多种不同的组合方式。
   3. Trie数据结构在中文文本中，由于其空间和时间效率的限制以及中文文本的复杂性，可能不是最佳选择。
3. 双数组Trie树是一种基于Trie树的数据结构
   1. DAT 的静态构建是通过将 Trie 树转换为两个数组实现的。其中一个数组存储 Trie 树中的节点信息，另一个数组则存储节点的孩子节点信息。这种实现方式具有很好的压缩性和查找效率。
   2. 当需要删除 Trie 树中的一个节点时，需要将该节点从 Trie 树中删除，并且可能需要对其祖先节点进行修改，以确保删除后的 Trie 树仍然是一棵有效的树。因为 DAT 中的数组是静态分配的，因此删除节点和修改数组可能会破坏原始数据结构的完整性，导致无法保证查找的正确性。
4. [AhoCorasickDoubleArrayTrie](https://www.hankcs.com/program/algorithm/aho-corasick-double-array-trie.html)
   1. 在实际的应用中，存储大辞典的话，会出现溢出情况，并且在issues中也有他人出现这种问题。
5. ***本文重点实现 FST 数据机构，详见代码***
   1. 使用[语料库](https://github.com/wainshine)分别加入 AhoCorasickDoubleArrayTrie 和 FST，前者出现溢出情况（400w数据量左右），后者正常使用。
   2. 可以动态增删关键词；

## 代码
```java
package com.pap.base.util.fst;

import java.io.Serializable;
import java.util.HashMap;

/**
 * FST
 */
public class FST implements Serializable {

    private HashMap<Character, FST> transitions = new HashMap<>();
    private boolean isFinalState = false;

    public void addWord(String word) {
        if (word.isEmpty()) {
            isFinalState = true;
            return;
        }
        char c = word.charAt(0);
        FST nextState = transitions.get(c);
        if (nextState == null) {
            nextState = new FST();
            transitions.put(c, nextState);
        }
        nextState.addWord(word.substring(1));
    }

    public boolean isWord(String word) {
        if (word.isEmpty()) {
            return isFinalState;
        }
        char c = word.charAt(0);
        FST nextState = transitions.get(c);
        if (nextState == null) {
            return false;
        }
        return nextState.isWord(word.substring(1));
    }

    public boolean removeWord(String word) {
        if (word.isEmpty()) {
            boolean wasFinal = isFinalState;
            isFinalState = false;
            return wasFinal;
        }
        char c = word.charAt(0);
        FST nextState = transitions.get(c);
        if (nextState == null) {
            return false;
        }
        boolean wasRemoved = nextState.removeWord(word.substring(1));
        if (nextState.transitions.isEmpty() && !nextState.isFinalState) {
            transitions.remove(c);
        }
        return wasRemoved;
    }

}

```

```java
package com.pap.base.util.fst;

import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;

/**
 * 工具类与测试方法
 */
public class FSTUtil {

    /**
     * 最大匹配，并且额外返回了字典在文本中所处的位置。
     *
     * @param text
     * @param dict
     * @return
     */
    public static List<ValueLocationDTO> maxMatchLocation(String text, FST dict) {
        List<ValueLocationDTO> result = new ArrayList<>();
        int start = 0;
        while (start < text.length()) {
            int end = text.length();
            while (end > start) {
                String substr = text.substring(start, end);
                if (dict.isWord(substr)) {
                    result.add(new ValueLocationDTO(substr, start, end));
                    start = end;
                    break;
                }
                end--;
            }
            if (end == start) {
                start++;
            }
        }
        return result;
    }

    public static void main(String[] args) throws Exception {
        FST dict = new FST();

        // 这里从字典表里面把数据取出来，数据来源： https://github.com/wainshine
        List<String> names = Files.readAllLines(Paths.get("C:\\Users\\86181\\Desktop\\Chinese_Names_Corpus（120W）.txt"));
        for (String name : names) {
            dict.addWord(name);
        }

        String text = "试一试分词效果，我得名字叫彭胜文，曾用名是彭胜利";
        List<ValueLocationDTO> result = maxMatchLocation(text, dict);
        System.out.println(result);

        dict.removeWord("彭胜文");
        dict.removeWord("彭胜");
        String text2 = "试一试分词效果，我得名字叫彭胜文，曾用名是彭胜利";
        List<ValueLocationDTO> result2 = maxMatchLocation(text2, dict);
        System.out.println(result2);
    }
}

```

```java
public class ValueLocationDTO implements Serializable {
    private String text;

    private Integer start;

    private Integer end;

    public ValueLocationDTO(String text, Integer start, Integer end) {
        this.text = text;
        this.start = start;
        this.end = end;
    }
}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
