# 动态规划-编辑距离-两字符串集合重排序

## 背景
&ensp;&ensp;近期遇到一个需求，想要对两个字符串集合进行重排序（对齐）操作，将两个字符串集合中尽可能相同的字符串存放到相同的位置上。

## 示例
&ensp;&ensp;假设有如下两个字符串集合，对两个集合进行重排序操作，由于 ArrayRight[0] 与 ArrayLeft[1] 最相似，那么重排序后的字符串集合中，ArrayLeft[1]应该处于首位。
1. ArrayLeft : ("班级", "姓名", "年级", "生日")
2. ArrayRight: ("学生姓名", "学生生日")

## 方案
&ensp;&ensp;第一种方案考虑对两个字符串集合求并集，然后将差异的部分进行添加，这种方案要求在集合中的字符串完全相等，但是某些业务场景下，同一个物体的描述可能有细微的差别，这种情况下此方案略显不足。

&ensp;&ensp;第二种方案考虑将编辑距离引入进行计算，对两个字符串集合进行双层循环，将字符串集合间的问题转换为字符串间的问题，这种方案解决了第一种方案的弊端。

## 执行效果

&ensp;&ensp;将如上的 ArrayLeft、ArrayRight 进行输入， 对左侧的字符串集合进行重排序，输出的结果为： ("姓名", "生日", "班级", "年级")。

## 算法

&ensp;&ensp; 其中 DistanceUtiil.editDistance 就是常见的 编辑距离的实现，本文不做详细介绍，可以单独进行搜索。

```java
public class StringAlignment {

    /**
     * 两个字符串集合进行重排序，将相似的放置到相同位置
     * @param leftList  左侧List Arrays.asList("班级", "姓名", "年级", "生日", "生生");
     * @param rightList 右侧List Arrays.asList("学生姓名", "学生生日");
     * @return  右侧的 rightList 不变，左侧的返回新的List集合  [姓名, 生日, 班级, 年级]
     */
    public static List<String> reOrderLeftList(List<String> leftList, List<String> rightList) {
        if(leftList != null && rightList != null
                && leftList.size() > 0 && rightList.size() > 0) {
            List<String> orderedList = new ArrayList<>();
            for(String right : rightList) {
                Integer minValue = Integer.MAX_VALUE;
                String minWord = null;
                for(String left : leftList) {
                    if(!orderedList.contains(left)) {
                        int distinctInt = DistanceUtiil.editDistance(left, right);
                        if(distinctInt < minValue) {
                            minValue = distinctInt;
                            minWord = left;
                        }
                    }
                }
                if(minWord != null) {
                    orderedList.add(minWord);
                }
            }

            // leftList 有可能小于 rightList， 最后再把没加进来的加进来
            for(String left : leftList) {
                if(!orderedList.contains(left)) {
                    orderedList.add(left);
                }
            }
            return orderedList;
        } else {
            return leftList;
        }
    }

    /**
     *  编辑距离 - 计算出两个字符串的变化过程，将 word1 转换为 word2 的时候，需要进行操作的步骤。
     * @param word1 字符串1，假设输入： horse
     * @param word2 字符串2，假设输入： ros
     * @return 输出 word1 到 word2 的变换过程 ： [Replace 'h' with 'r', Copy 'o', Delete 'r', Copy 's', Delete 'e']
     */
    public static List<String> calEditDistanceWays(String word1, String word2) {
        int m = word1.length();
        int n = word2.length();
        List<String> operations = new ArrayList<>();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 0; i <= m; i++) {
            for (int j = 0; j <= n; j++) {
                if (i == 0) {
                    dp[i][j] = j;
                } else if (j == 0) {
                    dp[i][j] = i;
                } else if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = 1 + Math.min(dp[i - 1][j - 1], Math.min(dp[i - 1][j], dp[i][j - 1]));
                }
            }
        }

        int i = m, j = n;
        while (i > 0 && j > 0) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                operations.add("Copy '" + word1.charAt(i - 1) + "'");
                i--;
                j--;
            } else if (dp[i][j] == dp[i - 1][j - 1] + 1) {
                operations.add("Replace '" + word1.charAt(i - 1) + "' with '" + word2.charAt(j - 1) + "'");
                i--;
                j--;
            } else if (dp[i][j] == dp[i - 1][j] + 1) {
                operations.add("Delete '" + word1.charAt(i - 1) + "'");
                i--;
            } else if (dp[i][j] == dp[i][j - 1] + 1) {
                operations.add("Insert '" + word2.charAt(j - 1) + "'");
                j--;
            }
        }

        while (i > 0) {
            operations.add("Delete '" + word1.charAt(i - 1) + "'");
            i--;
        }

        while (j > 0) {
            operations.add("Insert '" + word2.charAt(j - 1) + "'");
            j--;
        }

        Collections.reverse(operations);
        return operations;
    }
}
```

## 参考

1. https://blog.csdn.net/allway2/article/details/127889054
2. https://gitee.com/alexgaoyh/pap-base/blob/v1/src/test/java/com/pap/base/util/dynamic/DistanceUtiil.java
