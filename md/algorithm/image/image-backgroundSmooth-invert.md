## 背景

&ensp;&ensp;《图像处理-Java-背景色平滑/反色》，之前在编写过关于[图像边缘检测-去黑边](https://pap-docs.pap.net.cn/#/md/algorithm/image/remove-black-border)、[图像边缘检测-自动纠偏](https://pap-docs.pap.net.cn/#/md/algorithm/image/auto-correction)、[图像处理-锐化](https://pap-docs.pap.net.cn/#/md/algorithm/image/sharpening-prewitt-overlay)、[图像处理-Java-去噪/高斯模糊/套红](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-denoise-gaussianBlur-red)等一系列文章，接下来主要介绍图像背景色平滑功能，简要介绍图像反色的功能。

## 概述

&ensp;&ensp;使用JAVA语音实现，实现思路是通过识别背景色并将接近背景色的像素进行平均处理，以减少图像中颜色的突变。分为如下步骤：1、颜色计数：2、背景色查找：3、平滑处理：4、颜色接近判断：5、获取平均颜色：6、替换图像。

## 实现效果示例

&ensp;&ensp;首先输入图像仍保持不变，输出分别是： 背景色平滑(请注意四周的黑框，并且文字部分可以看出来未收背景色平滑影响)、反色。

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/01/18/UWhVov5g7EGdjbA.jpg" alt="图像处理:背景色平滑/反色-输入图像" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/19/mq4jQL3I6XOtls2.jpg" alt="图像处理:背景色平滑-输出图像" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/19/3OK2M5b7syo14vY.jpg" alt="图像处理:反色-输出图像" style="width: 30%;">
</div>

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/algorithm/image/img)

## 代码实现

```html
    public static BufferedImage smoothBackground2(BufferedImage originalImage, int colorThreshold) {
        int width = originalImage.getWidth();
        int height = originalImage.getHeight();

        DataBuffer dataBuffer = originalImage.getRaster().getDataBuffer();
        byte[] pixels = ((DataBufferByte) dataBuffer).getData();
        Map<Integer, Integer> colorCount = new HashMap<>();
        for (int i = 0; i < pixels.length; i += 3) {
            int color = ((pixels[i] & 0xFF) << 16) | ((pixels[i + 1] & 0xFF) << 8) | (pixels[i + 2] & 0xFF);
            colorCount.put(color, colorCount.getOrDefault(color, 0) + 1);
        }
        int backgroundColor = findBackgroundColor(colorCount);

        BufferedImage smoothedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                Color pixelColor = new Color(originalImage.getRGB(x, y));

                // 如果像素颜色接近于背景色，进行平滑处理
                if (isColorCloseToBackground(pixelColor, new Color(backgroundColor), colorThreshold)) {
                    Color averageColor = getAverageColor(originalImage, x, y, 3);
                    smoothedImage.setRGB(x, y, averageColor.getRGB());
                } else {
                    // 不是背景色的像素，保持原样
                    smoothedImage.setRGB(x, y, originalImage.getRGB(x, y));
                }
            }
        }

        return smoothedImage;
    }

    private static boolean isColorCloseToBackground(Color color, Color backgroundColor, int colorThreshold) {
        int redDiff = Math.abs(color.getRed() - backgroundColor.getRed());
        int greenDiff = Math.abs(color.getGreen() - backgroundColor.getGreen());
        int blueDiff = Math.abs(color.getBlue() - backgroundColor.getBlue());

        // 判断颜色是否接近背景色
        return redDiff < colorThreshold && greenDiff < colorThreshold && blueDiff < colorThreshold;
    }

    private static Color getAverageColor(BufferedImage image, int centerX, int centerY, int windowSize) {
        int totalRed = 0;
        int totalGreen = 0;
        int totalBlue = 0;
        int count = 0;

        for (int y = centerY - windowSize/2; y <= centerY + windowSize/2; y++) {
            for (int x = centerX - windowSize/2; x <= centerX + windowSize/2; x++) {
                if (x >= 0 && x < image.getWidth() && y >= 0 && y < image.getHeight()) {
                    // 获取当前像素的颜色值
                    Color pixelColor = new Color(image.getRGB(x, y));
                    totalRed += pixelColor.getRed();
                    totalGreen += pixelColor.getGreen();
                    totalBlue += pixelColor.getBlue();
                    count++;
                }
            }
        }

        // 计算平均颜色值
        int averageRed = totalRed / count;
        int averageGreen = totalGreen / count;
        int averageBlue = totalBlue / count;

        return new Color(averageRed, averageGreen, averageBlue);
    }
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
