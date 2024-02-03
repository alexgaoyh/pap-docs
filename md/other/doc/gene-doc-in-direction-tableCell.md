## 背景

&ensp;&ensp;《一种Java语言下生成竖版表格文档的方法》本文是基于Java语言，引入POI从而生成竖版表格文档的方法，文字方向是竖向，本文仅做单元测试方法的分享。

## 效果

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/gene-doc-in-direction-tableCell.png)

![gene-doc-in-direction-tableCell.png](https://s2.loli.net/2024/02/03/G3SkZM4aVmntC5b.jpg)

## 代码

```html
    @Test
    public void textDirectionTable() {
        try {
            XWPFDocument document = new XWPFDocument();

            XWPFTable table1 = document.createTable(1, 3);
            table1.setWidth(10000);

            table1.getRow(0).getCell(0).setText("第0列：这里是正文的信息，http://pap-doc.pap.net.cn , 这里是正文的信息，http://pap-doc.pap.net.cn , 这里是正文的信息，http://pap-doc.pap.net.cn , 这里是正文的信息，http://pap-doc.pap.net.cn , 这里是正文的信息，http://pap-doc.pap.net.cn , ");
            XWPFTableCell cell00 = table1.getRow(0).getCell(0);
            // 宽度
            CTTc ctTc00 = cell00.getCTTc();
            CTTcPr tcPr00 = ctTc00.addNewTcPr();
            CTTblWidth width00 = tcPr00.addNewTcW();
            width00.setType(STTblWidth.DXA);
            width00.setW(BigInteger.valueOf(2000));

            CTTextDirection textDirec00 = cell00.getCTTc().addNewTcPr().addNewTextDirection();
            textDirec00.setVal(STTextDirection.TB_LR_V);

            table1.getRow(0).getCell(1).setText("第1列：");
            XWPFTableCell cell01 = table1.getRow(0).getCell(1);
            CTTextDirection textDirec01 = cell01.getCTTc().addNewTcPr().addNewTextDirection();
            textDirec01.setVal(STTextDirection.TB_LR_V);

            table1.getRow(0).getCell(2).setText("第2列：");
            XWPFTableCell cell02 = table1.getRow(0).getCell(2);
            CTTextDirection textDirec02 = cell02.getCTTc().addNewTcPr().addNewTextDirection();
            textDirec02.setVal(STTextDirection.TB_LR_V);

            // 表格高度
            XWPFTableRow headerRow = table1.getRow(0);
            headerRow.setHeight(10000);

            // 无边框
            CTTbl ctTbl = table1.getCTTbl();
            CTTblPr tblpro = ctTbl.addNewTblPr();
            CTTblBorders tblBorders = tblpro.addNewTblBorders();
            tblBorders.addNewTop().setVal(STBorder.NONE);
            tblBorders.addNewBottom().setVal(STBorder.NONE);
            tblBorders.addNewLeft().setVal(STBorder.NONE);
            tblBorders.addNewRight().setVal(STBorder.NONE);
            tblBorders.addNewInsideH().setVal(STBorder.NONE);
            tblBorders.addNewInsideV().setVal(STBorder.NONE);

            document.setTable(document.getTables().size() - 1, table1);

            String filePath = "pap.docx";
            FileOutputStream out = new FileOutputStream(new File(filePath));
            document.write(out);

            out.close();
            document.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
