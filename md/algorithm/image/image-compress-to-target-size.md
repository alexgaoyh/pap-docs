## 背景

&ensp;&ensp;《图像处理-Java-指定大小压缩》，之前在编写过关于[图像边缘检测-去黑边](https://pap-docs.pap.net.cn/#/md/algorithm/image/remove-black-border)、[图像边缘检测-自动纠偏](https://pap-docs.pap.net.cn/#/md/algorithm/image/auto-correction)、[图像处理-锐化](https://pap-docs.pap.net.cn/#/md/algorithm/image/sharpening-prewitt-overlay)、[图像处理-去噪/高斯模糊/套红](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-denoise-gaussianBlur-red)、[图像处理-背景色平滑/反色](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-backgroundSmooth-invert)等一系列文章，接下来主要介绍图像压缩，将图像压缩至指定大小。

## 概述

&ensp;&ensp;使用JAVA语言实现，将给定的JPEG图像文件调整到指定的目标文件大小，使用循环，不断地调整图像质量，然后保存图像并检查文件大小是否达到目标大小。

## 代码实现

```java
package com.pap.base.util.image;

import javax.imageio.ImageIO;
import javax.imageio.ImageWriteParam;
import javax.imageio.ImageWriter;
import javax.imageio.stream.FileImageOutputStream;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.Iterator;

public class JPGCompressTest {

    // 输入图像绝对路径、输出图像绝对路径、期望的图像大小
    public static void convertJpg(String inputImagePath, String outputImagePath, double targetSizeBytes) throws IOException {
        File inputFile = new File(inputImagePath);
        File outputFile = new File(outputImagePath);

        long currentSize = inputFile.length();
        float quality = 0.5f;

        while (currentSize > targetSizeBytes && quality > 0.1f) {
            BufferedImage bufferedImage = ImageIO.read(inputFile);
            writeJpgWithQuality(bufferedImage, outputFile, quality);

            currentSize = outputFile.length();
            quality -= 0.005f;
        }

        while (currentSize < targetSizeBytes) {
            BufferedImage bufferedImage = ImageIO.read(inputFile);
            writeJpgWithQuality(bufferedImage, outputFile, quality);

            currentSize = outputFile.length();
            quality += 0.005f;
        }

    }

    private static void writeJpgWithQuality(BufferedImage image, File outputFile, float quality) throws IOException {
        Iterator<ImageWriter> writers = ImageIO.getImageWritersByFormatName("jpg");
        if (writers.hasNext()) {
            ImageWriter writer = writers.next();
            ImageWriteParam param = writer.getDefaultWriteParam();

            param.setCompressionMode(ImageWriteParam.MODE_EXPLICIT);
            param.setCompressionQuality(quality);

            try (FileImageOutputStream output = new FileImageOutputStream(outputFile)) {
                writer.setOutput(output);
                writer.write(null, new javax.imageio.IIOImage(image, null, null), param);
            } finally {
                writer.dispose();
            }
        }
    }
}
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
