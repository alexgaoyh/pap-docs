## 背景

&ensp;&ensp;《字符串操作-逗号分割字符串转树形结构》近期一直在进行中文领域开源数据集的预处理操作，其中有一系列方法完全可以抽离出来，本文就介绍了了一种将逗号分割的字符串集合转为树形结构的方式，用来将开源数据集中的平铺数据结构转为树形，将无层级结构的数据转为层级结构。

## 示例

```html
    public static List<String> gene() {
        List<String> originList = new ArrayList<>();
        originList.add("序一");
        originList.add("序二");
        originList.add("卷一,目一");
        originList.add("卷一,目二");
        originList.add("卷一,目三");
        originList.add("卷一,目四");
        originList.add("卷一,目五");
        originList.add("卷一,目五,子一");
        originList.add("卷二,录一");
        originList.add("卷二,录二");
        originList.add("卷二,录三");
        originList.add("卷二,录四");
        originList.add("卷二,录五");
        return originList;
    }
```

```json
[ 
  {"id":"序一","name":"序一","parentId":"-1"},
  {"id":"序二","name":"序二","parentId":"-1"},
  {"id":"卷一","name":"卷一","parentId":"-1",
      "children":[
          {"id":"目一","name":"目一","parentId":"卷一"},
          {"id":"目二","name":"目二","parentId":"卷一"},
          {"id":"目三","name":"目三","parentId":"卷一"},
          {"id":"目四","name":"目四","parentId":"卷一"},
          {"id":"目五","name":"目五","parentId":"卷一",
              "children":[
                  {"id":"子一","name":"子一","parentId":"目五"}
              ]
          }
      ]
  },
  {"id":"卷二","name":"卷二","parentId":"-1",
      "children":[
          {"id":"录一","name":"录一","parentId":"卷二"},
          {"id":"录二","name":"录二","parentId":"卷二"},
          {"id":"录三","name":"录三","parentId":"卷二"},
          {"id":"录四","name":"录四","parentId":"卷二"},
          {"id":"录五","name":"录五","parentId":"卷二"}
      ]
  }
]
```

## 思路

&ensp;&ensp;首先现将原始的字符串集合转为平铺形式的树形结构集合，之后将平铺的属性结构集合使用递归方式构造成树形结构。

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap-base/blob/v1/src/test/java/com/pap/base/util/tree/DouHaoToTreeUtil.java

