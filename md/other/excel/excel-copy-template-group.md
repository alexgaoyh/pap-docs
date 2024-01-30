## 背景

&ensp;&ensp;《Excel复杂表头按组按行复制》在ERP软件中将数据导出为复杂表头的Excel，本例采用模板替换的思路，先将所有单元格生成，之后进行单元格替换。

## 效果

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/excel-copy-template-group.png)

![excel-copy-template-group.png](https://s2.loli.net/2024/01/30/DHeroGsCXjYMRNW.jpg)

## 实现

```html
    /**
     * 指定 源excel、目标excel、目标excel的sheet名称，将 小于 offset 偏移量的行全部复制，接下来制定行数 withinGroupLength 的数据当成一组，复制为 numberOfGroup 组。
     * @param sourceFilePath    源/模板 xlsx 的绝对路径
     * @param destFilePath  目标xlsx的绝对路径
     * @param targetSheetName   目标sheet的名称
     * @param offset        8
     * @param withinGroupLength 9
     * @param numberOfGroup 2
     */
    public static void copyRowWithStyleInGroup(String sourceFilePath, String destFilePath, String targetSheetName, int offset, int withinGroupLength, int numberOfGroup) {
        try {
            FileInputStream fis = new FileInputStream(sourceFilePath);
            Workbook sourceWorkbook = WorkbookFactory.create(fis);
            Sheet sourceSheet = sourceWorkbook.getSheetAt(0);

            Workbook destWorkbook = WorkbookFactory.create(true);
            Sheet destSheet = destWorkbook.createSheet(targetSheetName);

            for(int offsetIdx = 0; offsetIdx < offset; offsetIdx++) {
                Row sourceRow = sourceSheet.getRow(offsetIdx);
                Row destRow = destSheet.createRow(offsetIdx);
                copyRowHeight(sourceRow, destRow);

                if (sourceRow != null) {
                    for (int i = 0; i < sourceRow.getLastCellNum(); i++) {
                        Cell sourceCell = sourceRow.getCell(i);
                        Cell destCell = destRow.createCell(i);

                        copyCellValue(sourceCell, destCell);
                        copyCellStyle(sourceCell, destCell, destWorkbook);
                        copyColumnWidth(sourceSheet, destSheet, i);
                    }
                }

                copyMergedRegions(sourceSheet, destSheet, offsetIdx, offsetIdx);
            }

            for(int numberOfGroupIdx = 1; numberOfGroupIdx <= numberOfGroup; numberOfGroupIdx++) {
                for(int withinGroupLengthIdx = 0; withinGroupLengthIdx < withinGroupLength; withinGroupLengthIdx++) {
                    int sourceRowIdx = offset + withinGroupLengthIdx;
                    int targetRowIdx = (offset) + (numberOfGroupIdx - 1) * withinGroupLength + withinGroupLengthIdx;

                    Row sourceRow = sourceSheet.getRow(sourceRowIdx);
                    Row destRow = destSheet.createRow(targetRowIdx);
                    copyRowHeight(sourceRow, destRow);

                    if (sourceRow != null) {
                        for (int i = 0; i < sourceRow.getLastCellNum(); i++) {
                            Cell sourceCell = sourceRow.getCell(i);
                            Cell destCell = destRow.createCell(i);

                            copyCellValue(sourceCell, destCell);
                            copyCellStyle(sourceCell, destCell, destWorkbook);
                            copyColumnWidth(sourceSheet, destSheet, i);
                        }
                    }

                    copyMergedRegions(sourceSheet, destSheet, sourceRowIdx, targetRowIdx);
                }
            }


            FileOutputStream fos = new FileOutputStream(destFilePath);
            destWorkbook.write(fos);

            fis.close();
            fos.close();
            sourceWorkbook.close();
            destWorkbook.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap4j-boot3/blob/master/pap4j-common/pap4j-common-excel/src/main/java/cn/net/pap/common/excel/ExcelCopyUtil.java
