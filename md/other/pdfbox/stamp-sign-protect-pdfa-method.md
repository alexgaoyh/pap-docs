## 背景

&ensp;&ensp;《针对PDF文档:印章、数字签名、编辑保护、PDF/A的Java工具类》本文是基于Java语言，引入POI从而提供将富文本编辑器内的html内容转换为docx的方式。

## 代码

&ensp;&ensp;引入pom坐标

```xml
    <dependency>
        <groupId>cn.net.pap</groupId>
        <artifactId>pap4j-common-pdf</artifactId>
        <version>0.0.1</version>
    </dependency>
```

&ensp;&ensp;测试方法

```html

    @Test
    public void pdfTest() {
        try {
            // 添加印章： 原始pdf、印章图像、输出pdf
            PDFUtil.addStamp("origin.pdf", "alexgaoyh.png", "output.pdf");

            // 添加禁止编辑(密码)： 原始pdf、ownerPassword、userPassword、输出pdf
            PDFUtil.addProtect("origin.pdf", "alexgaoyh", "pap.net", "output.pdf");

            // 添加数字签名： 原始pdf、*.p12文件、*.p12文件对应密码、输出pdf
            PDFUtil.addSign("origin.pdf", "alexgaoyh.p12", "alexgaoyh", "output.pdf");

            // 转换PDF/A标准： 原始pdf，输出pdf
            PDFUtil.convertPDFA("origin.pdf", "output.pdf");
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

```


## 总结

&ensp;&ensp;本方法依赖 pdfbox 3.0.2版本，并且引入bcprov-jdk15on、bcpkix-jdk15on处理数字签名中的证书部分。


## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap4j-boot3/
