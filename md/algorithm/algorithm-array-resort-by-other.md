## 概述

&ensp;&ensp;《字符串操作-两个数组之间的重排序》本文提供了一种更容易理解的两个数组之间的重排序方法。

## 背景

&ensp;&ensp;假设有一个不是很恰当的场景，目前有一个包含多个学生编号的数组，向成绩模块发送请求去获得对应的成绩，返回了数组长度一致的成绩信息，现在想要根据学生成绩去重排序学生编号。

## 原始思路

&ensp;&ensp;先对包含成绩的数组进行处理，记录相关信息，再对包含学生编号的数组进行处理。

&ensp;&ensp;原始方案能够解决此问题，但是一直觉得解决方案不够美，直到使用了一些API，最终美化了代码。

## 改进

&ensp;&ensp;定义了两个数组，第一个数组是学生编号数组集合，第二个数组是学生成绩数组集合，返回的是根据学生成绩降序的重排序的学生编号集合

```html
    public void test() {
        String[] studentArray = new String[]{"STU202001010001", "STU202001010002", "STU202001010003", "STU202001010004", "STU202001010005"};
        Integer[] scoreArray = new Integer[]{70, 80, 90, 60, 50};
        String[] sortedStudentArray = reSort(studentArray, scoreArray); // ["STU202001010003", "STU202001010002", "STU202001010001", "STU202001010004", "STU202001010005"]
    }

    public static String[] reSort(String[] originArray, Integer[] sortArray) {
        // 使用自定义比较器
        Integer[] indexes = new Integer[originArray.length];
        for (int i = 0; i < originArray.length; i++) {
            indexes[i] = i;
        }

        Arrays.sort(indexes, (a, b) -> sortArray[b].compareTo(sortArray[a]));

        // 重新排序学生编号数组
        String[] sortedArray = new String[originArray.length];
        for (int i = 0; i < originArray.length; i++) {
            sortedArray[i] = originArray[indexes[i]];
        }

        // 更新原数组
        System.arraycopy(sortedArray, 0, originArray, 0, originArray.length);
        return sortedArray;
    }
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
