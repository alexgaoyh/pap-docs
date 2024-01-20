## 概述

&ensp;&ensp;《思考-RBAC中对于权限编码部分的压缩处理》近段时间重新学习了一下压缩算法，突然想到在平时的程序设计中能否对其进行应用，进而想到最基础的RBAC权限设计，可否将大量的一对多的关联关系部分数据进行压缩，本文介绍只是一种思路，作者并未在真实环境对此进行应用。

## 背景

&ensp;&ensp;在最基础的RBAC权限设计中，有用户与角色的关联表和角色与权限的关联表，这两个关联表都是多对多的，极端情况下关联表的数据量会越来越大，可否将数据进行优化？是作者编写本文的一个背景思考。

&ensp;&ensp;在Linux系统中对于文件的权限就是用位计算实现的，需要控制读、写、执行3个权限，仅需用3位二进制数来表示，可否将位运算的思想加入到权限设计中。

## 实现

&ensp;&ensp;基于如上两个背景思考，开始了一些技术预演，包括位运算的一些示例和一个更好的压缩/解压算法，减少位向量在数据库中的存储。

&ensp;&ensp;代码如下，定义BitSet表示权限集合，并且对BitSet进行数据初始化，之后进行压缩和解压，并判断某一个权限编码是否在解压后的位向量中。

```java
package com.pap.base.util.bitset;

import org.junit.Test;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.Base64;
import java.util.BitSet;
import java.util.zip.GZIPInputStream;
import java.util.zip.GZIPOutputStream;

public class BitSetCompressTest {

    // @Test
    public void test() {
        // 创建一个 BitSet，表示权限集合
        BitSet permissionSet = new BitSet();
        for (int i = 0; i < 1000000; i++) {
            permissionSet.set(i);
        }

        // 将 BitSet 压缩为字符串
        String compressedString = compressBitSet(permissionSet);
        // 解压
        BitSet loadedPermissionSet = decompressBitSet(compressedString);

        int permissionToCheck = 888;
        if (loadedPermissionSet.get(permissionToCheck)) {
            System.out.println("权限存在: " + permissionToCheck);
        } else {
            System.out.println("权限不存在: " + permissionToCheck);
        }
    }

    // 将 BitSet 压缩为 GZIP 压缩后再进行 Base64 编码的字符串
    private static String compressBitSet(BitSet bitSet) {
        byte[] byteArray = bitSet.toByteArray();
        try {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            try (GZIPOutputStream gzipOutputStream = new GZIPOutputStream(baos)) {
                gzipOutputStream.write(byteArray);
            }
            return Base64.getEncoder().encodeToString(baos.toByteArray());
        } catch (IOException e) {
            e.printStackTrace();
            return "";
        }
    }

    // 将 GZIP 压缩后的 Base64 编码的字符串解压为 BitSet
    private static BitSet decompressBitSet(String compressedString) {
        byte[] compressedByteArray = Base64.getDecoder().decode(compressedString);
        try (ByteArrayInputStream bais = new ByteArrayInputStream(compressedByteArray);
             GZIPInputStream gzipInputStream = new GZIPInputStream(bais)) {
            byte[] buffer = new byte[1024];
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int len;
            while ((len = gzipInputStream.read(buffer)) != -1) {
                baos.write(buffer, 0, len);
            }
            return BitSet.valueOf(baos.toByteArray());
        } catch (IOException e) {
            e.printStackTrace();
            return new BitSet();
        }
    }

}
```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
