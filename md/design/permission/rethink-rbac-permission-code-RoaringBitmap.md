## 概述

&ensp;&ensp;《思考-RBAC中对于权限编码部分的压缩处理(RoaringBitmap)》前期编写了[思考-RBAC中对于权限编码部分的压缩处理](https://pap-docs.pap.net.cn/#/md/design/permission/rethink-rbac-permission-code)，其中关于Bitmap的部分近期又做了思考，经过资料查阅，本文针对 RoaringBitmap 进行概述。

## 背景

&ensp;&ensp;RoaringBitmap是一种进行压缩位图的数据结构，减少内存占用，能够满足Bitmap的绝大部分基本操作。

&ensp;&ensp;RoaringBitmap广泛应用：[ElasticSearch](https://www.elastic.co/cn/blog/frame-of-reference-and-roaring-bitmaps)等。

&ensp;&ensp;如下测试代码，使用Roaring64NavigableMap添加了百万条数据，并且进行了取值、判断数据长度、序列化/反序列化等操作。

```java
package cn.net.pap.common.bitmap;

import org.junit.jupiter.api.Test;
import org.roaringbitmap.longlong.LongIterator;
import org.roaringbitmap.longlong.Roaring64NavigableMap;
import static org.junit.jupiter.api.Assertions.assertTrue;

public class Roaring64NavigableMapTest {
    @Test
    public void simpleTest() {
        long currentTimeMillis = System.currentTimeMillis();
        Roaring64NavigableMap r64nMap = new Roaring64NavigableMap();
        // 添加范围，前闭区间后开区间
        long rangeLong = 1000000l;
        r64nMap.add(currentTimeMillis, currentTimeMillis + rangeLong);
        // 取第K个值，并判断值是否正确。
        long value1000 = r64nMap.select(1000);
        assertTrue(currentTimeMillis == value1000 - 1000l);
        // 判断数据长度
        assertTrue(r64nMap.getLongCardinality() == rangeLong);
        // 获得所有值
        LongIterator longIterator = r64nMap.getLongIterator();
        while (longIterator.hasNext()){
            assertTrue(longIterator.next() == currentTimeMillis);
            break;
        }

        // 序列化
        String serialize = Roaring64NavigableMapUtil.serialize(r64nMap);
        // 反序列化
        Roaring64NavigableMap dBitMap = Roaring64NavigableMapUtil.deserialize(serialize);
        // 经过序列化和反序列化之后，再次判断数据长度
        assertTrue(dBitMap.getLongCardinality() == rangeLong);
    }

}

```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
