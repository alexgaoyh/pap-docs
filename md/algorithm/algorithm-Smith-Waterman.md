# 动态规划-序列比对-Smith-Waterman

## 背景
&ensp;&ensp;近期遇到一些关于字符串比对的业务需求，并通过[编辑距离](https://pap-docs.pap.net.cn/#/md/algorithm/algorithm-two-str-list-reorder)进行了实现，并达到了业务要求。基于类似的业务场景，作者思考有没有其他的方案：故了解Smith-Waterman 算法并进行本文的编写。

## 示例
&ensp;&ensp;假设有如下两个字符串，对这两个字符串进行比对，并设置一系列的参数，可以得到一系列相似子串。

&ensp;&ensp;输入两个字符串，在限制参数的情况下，输出三组相似的子串，其中-代表差异。

1. **input.sequence1**: AACGTACTCAAGTCT
2. **input.sequence2**: TCGTACTCTAACGAT
3. **output.list[0]**:  CGTACTCCAA->CGTACTC-AA
4. **output.list[1]**:  CGTACTCCAAAG->CGTACTC-AA-G
5. **output.list[2]**:  CGTACTCCAAAGGT->CGTACTC-AA-G-T


## 算法

```java
package com.pap.base.util.string;

import java.util.ArrayList;
import java.util.List;

public class SmithWaterman {

    /**
     * List<Pair<String, String>> alignments = smithWaterman(sequence1, sequence2, matchScore, mismatchScore, gapPenalty);
     *
     * @param sequence1     AACGTACTCAAGTCT
     * @param sequence2     TCGTACTCTAACGAT
     * @param matchScore    2
     * @param mismatchScore -1
     * @param gapPenalty    -2
     * @return ["CGTACTCCAA : CGTACTC-AA",  "CGTACTCCAAAG : CGTACTC-AA-G",  "CGTACTCCAAAGGT : CGTACTC-AA-G-T"]
     */
    public static List<Pair<String, String>> smithWaterman(String sequence1, String sequence2, int matchScore, int mismatchScore, int gapPenalty) {
        int m = sequence1.length();
        int n = sequence2.length();

        int[][] matrix = new int[m + 1][n + 1];
        List<Pair<String, String>> alignments = new ArrayList<>();
        int maxScore = 0;

        for (int i = 0; i <= m; i++) {
            matrix[i][0] = 0;
        }

        for (int j = 0; j <= n; j++) {
            matrix[0][j] = 0;
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                int match = matrix[i - 1][j - 1] + (sequence1.charAt(i - 1) == sequence2.charAt(j - 1) ? matchScore : mismatchScore);
                int delete = matrix[i - 1][j] + gapPenalty;
                int insert = matrix[i][j - 1] + gapPenalty;

                matrix[i][j] = Math.max(0, Math.max(match, Math.max(delete, insert)));

                if (matrix[i][j] >= maxScore) {
                    if (matrix[i][j] > maxScore) {
                        maxScore = matrix[i][j];
                        alignments.clear();
                    }
                    Pair<String, String> alignment = backtrack(matrix, sequence1, sequence2, i, j, "", "", matchScore, mismatchScore);
                    alignments.add(alignment);
                }
            }
        }

        return alignments;
    }

    private static Pair<String, String> backtrack(int[][] matrix, String sequence1, String sequence2, int i, int j, String currentAlignment1, String currentAlignment2, int matchScore, int mismatchScore) {
        if (i == 0 || j == 0 || matrix[i][j] == 0) {
            return new Pair<>(currentAlignment1, currentAlignment2);
        }

        if (matrix[i][j] == matrix[i - 1][j - 1] + (sequence1.charAt(i - 1) == sequence2.charAt(j - 1) ? matchScore : mismatchScore)) {
            return backtrack(matrix, sequence1, sequence2, i - 1, j - 1, sequence1.charAt(i - 1) + currentAlignment1, sequence2.charAt(j - 1) + currentAlignment2, matchScore, mismatchScore);
        }

        if (matrix[i][j] == matrix[i - 1][j] - 2) {
            return backtrack(matrix, sequence1, sequence2, i - 1, j, "-" + currentAlignment1, sequence2.charAt(j - 1) + currentAlignment2, matchScore, mismatchScore);
        }

        if (matrix[i][j] == matrix[i][j - 1] - 2) {
            return backtrack(matrix, sequence1, sequence2, i, j - 1, sequence1.charAt(i - 1) + currentAlignment1, "-" + currentAlignment2, matchScore, mismatchScore);
        }

        return null;
    }

    static class Pair<K, V> {
        private K key;
        private V value;

        public Pair(K key, V value) {
            this.key = key;
            this.value = value;
        }

        public K getKey() {
            return key;
        }

        public V getValue() {
            return value;
        }

        public void setKey(K key) {
            this.key = key;
        }

        public void setValue(V value) {
            this.value = value;
        }

        @Override
        public String toString() {
            return "(" + key + ", " + value + ")";
        }

    }
}

```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
