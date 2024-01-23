# 动态规划-序列比对-最长公共子序列

## 背景
&ensp;&ensp;近期遇到一些关于字符串比对的业务需求，并通过[编辑距离](https://pap-docs.pap.net.cn/#/md/algorithm/algorithm-two-str-list-reorder)、[Smith-Waterman](https://pap-docs.pap.net.cn/#/md/algorithm/algorithm-Smith-Waterman)进行了实现，并达到了业务要求。基于类似的业务场景，作者思考有没有其他的方案：故了解最长公共子序列LCS算法并进行本文的编写。

## 示例
&ensp;&ensp;假设有如下两个字符串，对这两个字符串进行比对，得到最长公共子序列，并且包含每个字符在原始字符串中的位置。

&ensp;&ensp;输入两个字符串ABCBDAB和BDCAB，输出最长公共子序列BDAB，并且最长公共子序列在左侧字符串中的位置是[2,5,6,7]，在右侧字符串中的位置是[1,2,4,5]。

1. **input.sequence1**: ABCBDAB
2. **input.sequence2**: BDCAB
3. **output.LCS**:  BDAB
4. **output.position-sequence1**:  2 5 6 7
5. **output.position-sequence2**:  1 2 4 5


## 算法

```java
package com.pap.base.util.string;

public class LongestCommonSubsequence {

    public static void main(String[] args) {
        String str1 = "ABCBDAB";
        String str2 = "BDCAB";

        int[][] dp = computeLCS(str1, str2);

        String lcs = findLCS(dp, str1, str2);

        System.out.println("Longest Common Subsequence: " + lcs);
        System.out.println("Positions in String-1 : " + getLCSPositions(dp, str1, lcs));
        System.out.println("Positions in String-2 : " + getLCSPositions(dp, str2, lcs));
    }

    private static int[][] computeLCS(String str1, String str2) {
        int m = str1.length();
        int n = str2.length();

        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (str1.charAt(i - 1) == str2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp;
    }

    private static String findLCS(int[][] dp, String str1, String str2) {
        int m = str1.length();
        int n = str2.length();

        StringBuilder lcs = new StringBuilder();
        int i = m, j = n;

        while (i > 0 && j > 0) {
            if (str1.charAt(i - 1) == str2.charAt(j - 1)) {
                lcs.insert(0, str1.charAt(i - 1));
                i--;
                j--;
            } else if (dp[i - 1][j] > dp[i][j - 1]) {
                i--;
            } else {
                j--;
            }
        }

        return lcs.toString();
    }

    private static String getLCSPositions(int[][] dp, String str, String lcs) {
        StringBuilder positions = new StringBuilder();
        int i = 1, j = 1;

        for (char c : lcs.toCharArray()) {
            while (i <= str.length() && str.charAt(i - 1) != c) {
                i++;
            }

            positions.append(i).append(" ");
            i++;
        }

        return positions.toString().trim();
    }
}
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
