## 概述

&ensp;&ensp;《集合-Java-笛卡尔积、平铺》本文是关于对集合进行笛卡尔积操作、平铺操作的示例，技术层面并不复杂，故不过多进行介绍。

## 背景

&ensp;&ensp;笛卡尔积： 每个业务键有多个值，把所有值的集合都计算出来。可以类比关系型数据库的关联查询。

&ensp;&ensp;平铺： 每个业务键有多个值，把所有值的集合都计算出来，并区分当前选择的是哪个业务键。

## 实现

&ensp;&ensp;递归实现笛卡尔积，如下代码可以直接执行。

```html
    public void test() {
        List<List<String>> strList = new ArrayList<>();
        for(int i = 1; i <= 2; i++) {
            List<String> partStrList = new ArrayList<>();
            for(int j = 1; j <= 2; j++) {
                partStrList.add("(" + (i + "") + (j + "") + ")");
            }
            strList.add(partStrList);
        }

        List<String> descartesList = descartes(strList, 0, StringUtils.EMPTY, new ArrayList<>()); 
        // [(11)(21), (11)(22), (12)(21), (12)(22)]
    }

    public static List<String> descartes(List<List<String>> dimensionValues, int position, String originCode,
                                   List<String> result) {
        // 获取指定行数据
        List<String> rowValue = dimensionValues.get(position);
        for (String code : rowValue) {
            // 第一行不用拼接，直接copy，第二行开始需要在末尾拼接code
            String resultCode = position == 0 ? code : originCode + code;
            // 如果当前位置是最后一行，则可以添加最终resultCode构建结果
            if (position == dimensionValues.size() - 1) {
                result.add(resultCode);
            } else {
                // 否则进入下一行
                descartes(dimensionValues, position + 1, resultCode, result);
            }
        }
        return result;
    }
```

&ensp;&ensp;由于实际应用场景中，在数据转平铺的过程中，有时候需要记录数据值对应的是哪个业务键，故增加如下的工具类。

```html
    public void test() {
        Map<String, List<String>> inputMap = new LinkedHashMap<>();

        List<String> key1List = Arrays.asList(new String[]{"00001"});
        inputMap.put("key1", key1List);

        List<String> key2List = Arrays.asList(new String[]{"00001", "00002"});
        inputMap.put("key2", key2List);

        List<String> key3List = Arrays.asList(new String[]{"00001", "00002", "00003"});
        inputMap.put("key3", key3List);

        List<Map<String, String>> flatList = flatMap(inputMap);
        // [{key1=00001, key2=00001, key3=00001}, {key1=00001, key2=00002, key3=00001}, {key1=00001, key2=00001, key3=00002}, {key1=00001, key2=00002, key3=00002}, {key1=00001, key2=00001, key3=00003}, {key1=00001, key2=00002, key3=00003}]
    }

    public static List<Map<String, String>> flatMap(Map<String, List<String>> inputMap) {
        // 类似笛卡尔积的操作，转平铺
        List<Map<String, String>> flatList = new ArrayList<>();
        // 使用Stream进行转换
        inputMap.forEach((key, values) -> {
            List<Map<String, String>> intermediateList = new ArrayList<>();

            // 为每个值创建一个新的Map，并与之前的结果进行笛卡尔积
            values.forEach(value -> {
                if (flatList.isEmpty()) {
                    Map<String, String> newMap = new HashMap<>();
                    newMap.put(key, value);
                    intermediateList.add(newMap);
                } else {
                    flatList.forEach(existingMap -> {
                        Map<String, String> newMap = new HashMap<>(existingMap);
                        newMap.put(key, value);
                        intermediateList.add(newMap);
                    });
                }
            });
            flatList.clear();
            flatList.addAll(intermediateList);
        });
        return flatList;
    }
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
