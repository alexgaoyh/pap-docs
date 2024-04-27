## 背景

&ensp;&ensp;《EasyExcel导出-自适应图像尺寸》最近在使用EasyExcel进行图像数据导出的时候，需要在单元格内自适应缩放图像，防止图像被拉伸。。

## 效果

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/easyexcel-export-image.png)

![easyexcel-export-image.jpg](https://s2.loli.net/2024/04/27/Rp312alc6tTIune.jpg)

## 实现

```java
package cn.net.pap.common.excel.handle;

import com.alibaba.excel.metadata.Head;
import com.alibaba.excel.metadata.data.ImageData;
import com.alibaba.excel.metadata.data.WriteCellData;
import com.alibaba.excel.write.handler.CellWriteHandler;
import com.alibaba.excel.write.metadata.holder.WriteSheetHolder;
import com.alibaba.excel.write.metadata.holder.WriteTableHolder;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.util.CellRangeAddress;

import javax.imageio.ImageIO;
import java.io.ByteArrayInputStream;
import java.util.List;
import java.util.Objects;

public class ImageModifyHandler implements CellWriteHandler {

    // 256分之一的字符宽度转换为标准字符宽度。
    public static Integer standardCharacterWidth = 256;

    // 7.5是一个估算的字符到像素的转换因子
    public static Float character2PixelFactor = 7.5f;

    // 将点转换为英寸，因为1点 = 1/72英寸。
    public static Integer pixel2InchFactor = 72;

    // 英寸转换为像素，其中96是常用的DPI（每英寸像素数）值。
    public static Integer dpi = 96;

    // 行高与像素的转换因子
    public static Float rowHeight2PixelFactor = 1.3333f;


    /**
     * 后单元格数据转换
     *
     * @param writeSheetHolder 写单夹
     * @param writeTableHolder 写表夹
     * @param cellData         单元格数据
     * @param cell             细胞
     * @param head             头
     * @param relativeRowIndex 相对行索引
     * @param isHead           是头
     */
    @Override
    public void afterCellDataConverted(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder,
                                       WriteCellData<?> cellData, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {
        boolean noImageValue = Objects.isNull(cellData) || cellData.getImageDataList() == null || cellData.getImageDataList().size() == 0;
        if (Objects.equals(Boolean.TRUE, isHead) || noImageValue) {
            return;
        }
        Sheet sheet = cell.getSheet();
        int mergeColumNum = getMergeColumNum(cell, sheet);
        int mergeRowNum = getMergeRowNum(cell, sheet);
        ImageData imageData = cellData.getImageDataList().get(0);
        imageData.setRelativeLastRowIndex(mergeRowNum - 1);
        imageData.setRelativeLastColumnIndex(mergeColumNum - 1);

        // 处理图像缩放 - 等比例缩放
        try {
            ByteArrayInputStream bis = new ByteArrayInputStream(imageData.getImage());
            java.awt.image.BufferedImage image = ImageIO.read(bis);
            int targetWidth = (int)(sheet.getColumnWidth(cell.getColumnIndex()) / standardCharacterWidth * character2PixelFactor);
            int targetHeight = (int)(cell.getRow().getHeightInPoints() / pixel2InchFactor * dpi);

            // 计算图像的缩放比例
            double scaleX = (double) targetWidth / image.getWidth();
            double scaleY = (double) targetHeight / image.getHeight();
            double scale = Math.min(scaleX, scaleY);

            // 计算缩放后的图像大小
            int scaledWidth = (int) (image.getWidth() * scale);
            int scaledHeight = (int) (image.getHeight() * scale);

            // 计算上下左右四个角的空白
            int topPadding = (targetHeight - scaledHeight) / 2;
            int bottomPadding = targetHeight - scaledHeight - topPadding;
            int leftPadding = (targetWidth - scaledWidth) / 2;
            int rightPadding = targetWidth - scaledWidth - leftPadding;

            // 行高（点）= 像素高度 / 1.3333
            imageData.setTop((int)(topPadding/rowHeight2PixelFactor));
            imageData.setBottom((int)(bottomPadding/rowHeight2PixelFactor));
            imageData.setLeft((int)(leftPadding/rowHeight2PixelFactor));
            imageData.setRight((int)(rightPadding/rowHeight2PixelFactor));

            bis.close();
        } catch (Exception e) {
        }

        CellWriteHandler.super.afterCellDataConverted(writeSheetHolder, writeTableHolder, cellData, cell, head, relativeRowIndex, isHead);
    }


    /**
     * 得到合并行num
     *
     * @param cell  细胞
     * @param sheet 表
     * @return int
     */
    public static int getMergeRowNum(Cell cell, Sheet sheet) {
        int mergeSize = 1;
        List<CellRangeAddress> mergedRegions = sheet.getMergedRegions();
        for (CellRangeAddress cellRangeAddress : mergedRegions) {
            if (cellRangeAddress.isInRange(cell)) {
                //获取合并的行数
                mergeSize = cellRangeAddress.getLastRow() - cellRangeAddress.getFirstRow() + 1;
                break;
            }
        }
        return mergeSize;
    }

    /**
     * 得到合并列num
     *
     * @param cell  细胞
     * @param sheet 表
     * @return int
     */
    public static int getMergeColumNum(Cell cell, Sheet sheet) {
        int mergeSize = 1;
        List<CellRangeAddress> mergedRegions = sheet.getMergedRegions();
        for (CellRangeAddress cellRangeAddress : mergedRegions) {
            if (cellRangeAddress.isInRange(cell)) {
                //获取合并的列数
                mergeSize = cellRangeAddress.getLastColumn() - cellRangeAddress.getFirstColumn() + 1;
                break;
            }
        }
        return mergeSize;
    }

}


```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/pap4j-boot3/blob/master/pap4j-common/pap4j-common-excel/src/main/java/cn/net/pap/common/excel/handle/ImageModifyHandler.java
