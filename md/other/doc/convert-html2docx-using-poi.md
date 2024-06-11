## 背景

&ensp;&ensp;《转换Html(富文本编辑器)到docx的Java工具类》本文是基于Java语言，引入POI从而提供将富文本编辑器内的html内容转换为docx的方式。

## 效果

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/doc/img/html2docx.jpg)

![html2docx.jpg](https://s2.loli.net/2024/06/11/WMd9t7K3cYfGLbO.jpg)

## 代码

&ensp;&ensp;引入pom坐标

```xml
    <dependency>
        <groupId>cn.net.pap</groupId>
        <artifactId>pap4j-common-docx</artifactId>
        <version>0.0.1</version>
    </dependency>
```

&ensp;&ensp;测试方法

```html
    @Test
    public void html2DocxTest() {
        // 这里是从富文本编辑器里面获得的一个 html。
        String editorHTML = "<p><font color=\"#067d17\">pap.net.cn&nbsp; &nbsp;内容1&nbsp; &nbsp;</font><span style=\"color: rgb(6, 125, 23);\">pap.net.cn</span><span style=\"color: rgb(6, 125, 23);\">&nbsp;</span></p><p><font color=\"#067d17\">内容2&nbsp;</font><img src=\"https://foruda.gitee.com/avatar/1676898910937495644/73661_alexgaoyh_1578916342.png!avatar200\" style=\"width: 200px;\"><font color=\"#067d17\"> 内容3</font></p><p><span style=\"color: rgb(6, 125, 23);\">pap.net.cn</span><span style=\"color: rgb(6, 125, 23);\">&nbsp; &nbsp;</span><font color=\"#067d17\">内容4&nbsp; &nbsp;</font><span style=\"color: rgb(6, 125, 23);\">pap.net.cn</span><span style=\"color: rgb(6, 125, 23);\">&nbsp;</span></p>";
        boolean b = Html2DocxUtils.html2docx2UsingPOI(new StringBuffer(editorHTML), "OUT.docx");
        assertTrue(b);
    }
```

## 对比

&ensp;&ensp;在实际开发过程中，最开始计划使用 documents4j 进行转换，但是此方法依赖 Microsoft office，客户环境有可能千奇百怪，故在实际线上环境部署过程中，不再使用此方法，而改为使用poi进行转换处理。

```html
    IConverter converter = LocalConverter.builder().build();
    converter.convert(htmlInputStream).as(DocumentType.HTML).to(outputStream).as(DocumentType.DOCX).execute();
    converter.shutDown();
```

## 总结

&ensp;&ensp;本方法依赖 poi 4.1.2版本，支持HTML中存在三方的图像URL。


## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap4j-boot3/
