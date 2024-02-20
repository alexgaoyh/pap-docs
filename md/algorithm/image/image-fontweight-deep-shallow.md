## 背景

&ensp;&ensp;《图像处理-Java-字深字浅》，之前在编写过关于[图像边缘检测-去黑边](https://pap-docs.pap.net.cn/#/md/algorithm/image/remove-black-border)、[图像边缘检测-自动纠偏](https://pap-docs.pap.net.cn/#/md/algorithm/image/auto-correction)、[图像处理-锐化](https://pap-docs.pap.net.cn/#/md/algorithm/image/sharpening-prewitt-overlay)、[图像处理-去噪/高斯模糊/套红](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-denoise-gaussianBlur-red)、[图像处理-背景色平滑/反色](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-backgroundSmooth-invert)、[图像处理-指定大小压缩](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-compress-to-target-size)等一系列文章，接下来主要介绍图像字体变深/变浅操作。

## 概述

&ensp;&ensp;使用JAVA语言实现，实现思路是获得图像的边缘，判断边缘部分与白色像素点的相似度，对得到的边缘像素点在原始图像上进行像素点的调整(添加/减少)，从而实现边缘部分的变深/变浅操作。

## 实现效果示例

&ensp;&ensp;首先输入图像仍保持不变，输出分别是： 原图、字深图像、字浅图像。

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/02/19/AoXNsP7438ygGEh.jpg" alt="图像处理:输入图像" style="width: 30%;">
    <img src="https://s2.loli.net/2024/02/19/jSfw3zsiQ8KZdY4.jpg" alt="图像处理:字深" style="width: 30%;">
    <img src="https://s2.loli.net/2024/02/19/AbxsltF8CN26B7G.jpg" alt="图像处理:字浅" style="width: 30%;">
</div>

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/algorithm/image/img)

## 代码实现

```html

    /**
     * 在使用Prewitt算子获得图像边缘后，你可以将边缘信息与原图进行叠加处理来处理图像。
     * @param originalImage
     * @param threshold 阈值 30
     * @param scope 字深字浅幅度 : 字浅(threshold=30 && scope=60)、字深(threshold=30 && scope=-60)
     * @return
     */
    public static BufferedImage prewitterWithFontWeight(BufferedImage originalImage, int threshold, int scope) {
        // 获得Prewitt边缘图
        BufferedImage edgeImage = applyPrewittOperator(originalImage);
        // 将边缘图与原图叠加
        BufferedImage overlayImages = overlayFontWeight(originalImage, edgeImage, threshold, scope);
        return overlayImages;
    }

    private static BufferedImage overlayFontWeight(BufferedImage originalImage, BufferedImage edgeImage, int threshold, int scope) {
        int width = originalImage.getWidth();
        int height = originalImage.getHeight();

        BufferedImage resultImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                // 获取原图和边缘图的像素值
                int originalPixel = originalImage.getRGB(x, y);
                int edgePixel = edgeImage.getRGB(x, y);
                // 将边缘图的灰度值作为权重，这里简化为取Red分量
                int edgeWeight = (edgePixel >> 16) & 0xFF;

                // 忽略较小的边缘信息（根据阈值设定）
                if (edgeWeight < threshold) {
                    resultImage.setRGB(x, y, originalPixel);
                } else {
                    // 将原图和边缘图叠加，这里可以根据需求进行调整
                    int combinedRed = Math.min(255, ((originalPixel >> 16) & 0xFF) + scope);
                    int combinedGreen = Math.min(255, ((originalPixel >> 8) & 0xFF) + scope);
                    int combinedBlue = Math.min(255, (originalPixel & 0xFF) + scope);

                    // 合并颜色值
                    int combinedPixel = (combinedRed << 16) | (combinedGreen << 8) | combinedBlue;

                    // 设置叠加后的像素值
                    resultImage.setRGB(x, y, combinedPixel);
                }
            }
        }

        return resultImage;
    }

    private static BufferedImage applyPrewittOperator(BufferedImage originalImage) {
        int width = originalImage.getWidth();
        int height = originalImage.getHeight();

        BufferedImage sharpenedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

        // 定义Prewitt水平和垂直算子
        int[][] horizontalOperator = {
                {-1, 0, 1},
                {-1, 0, 1},
                {-1, 0, 1}
        };

        int[][] verticalOperator = {
                {-1, -1, -1},
                { 0,  0,  0},
                { 1,  1,  1}
        };

        // 应用水平Prewitt算子
        BufferedImage horizontalResult = applyConvolution(originalImage, horizontalOperator);
        // 应用垂直Prewitt算子
        BufferedImage verticalResult = applyConvolution(originalImage, verticalOperator);

        // 合并水平和垂直结果
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                int horizontalPixel = horizontalResult.getRGB(x, y);
                int verticalPixel = verticalResult.getRGB(x, y);

                // 取水平和垂直梯度的平方和的平方根
                int combinedPixel = combinePixels(horizontalPixel, verticalPixel);
                sharpenedImage.setRGB(x, y, combinedPixel);
            }
        }

        return sharpenedImage;
    }

    private static BufferedImage applyConvolution(BufferedImage image, int[][] operator) {
        int width = image.getWidth();
        int height = image.getHeight();

        BufferedImage result = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

        int operatorSize = operator.length;
        int offset = operatorSize / 2;

        // 归一化权重
        int weightSum = 0;
        for (int[] row : operator) {
            for (int value : row) {
                weightSum += Math.abs(value);
            }
        }

        if (weightSum == 0) {
            weightSum = 1; // 防止除以零错误
        }

        for (int y = offset; y < height - offset; y++) {
            for (int x = offset; x < width - offset; x++) {
                int sumRed = 0;
                int sumGreen = 0;
                int sumBlue = 0;

                for (int i = 0; i < operatorSize; i++) {
                    for (int j = 0; j < operatorSize; j++) {
                        int pixel = image.getRGB(x - offset + i, y - offset + j);
                        int weight = operator[i][j];

                        sumRed += weight * ((pixel >> 16) & 0xFF);
                        sumGreen += weight * ((pixel >> 8) & 0xFF);
                        sumBlue += weight * (pixel & 0xFF);
                    }
                }

                // 归一化颜色值
                sumRed = Math.min(255, Math.max(0, sumRed / weightSum));
                sumGreen = Math.min(255, Math.max(0, sumGreen / weightSum));
                sumBlue = Math.min(255, Math.max(0, sumBlue / weightSum));

                int combinedPixel = (sumRed << 16) | (sumGreen << 8) | sumBlue;
                result.setRGB(x, y, combinedPixel);
            }
        }

        return result;
    }

    private static int combinePixels(int pixel1, int pixel2) {
        int red1 = (pixel1 >> 16) & 0xFF;
        int green1 = (pixel1 >> 8) & 0xFF;
        int blue1 = pixel1 & 0xFF;

        int red2 = (pixel2 >> 16) & 0xFF;
        int green2 = (pixel2 >> 8) & 0xFF;
        int blue2 = pixel2 & 0xFF;

        int combinedRed = (int) Math.sqrt(red1 * red1 + red2 * red2);
        int combinedGreen = (int) Math.sqrt(green1 * green1 + green2 * green2);
        int combinedBlue = (int) Math.sqrt(blue1 * blue1 + blue2 * blue2);

        return (combinedRed << 16) | (combinedGreen << 8) | combinedBlue;
    }
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
