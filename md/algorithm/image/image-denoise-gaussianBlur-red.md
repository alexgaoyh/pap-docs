## 背景

&ensp;&ensp;《图像处理-Java-去噪/高斯模糊/套红》，之前在编写过关于[图像边缘检测-去黑边](https://pap-docs.pap.net.cn/#/md/algorithm/image/remove-black-border)、[图像边缘检测-自动纠偏](https://pap-docs.pap.net.cn/#/md/algorithm/image/auto-correction)、[图像处理-锐化](https://pap-docs.pap.net.cn/#/md/algorithm/image/sharpening-prewitt-overlay)等一系列文章，接下来主要介绍图像去噪，简要介绍高斯模糊和套红的功能。

## 概述

&ensp;&ensp;使用JAVA语音实现，实现思路是非局部均值（Non-Local Means, NLM）去噪方法。通过在图像中寻找相似的像素块，并使用加权平均的方式对这些块进行处理，从而达到去除噪声的效果。

## 实现效果示例

&ensp;&ensp;首先输入图像仍保持不变，请注意输入图像的左上角有人为添加的一个噪点。输出分别是： 去噪、高斯模糊、套红

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/01/18/UWhVov5g7EGdjbA.jpg" alt="图像处理:去噪/高斯模糊/套红-输入图像" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/18/OzIgPGp6xrVLJWK.jpg" alt="图像处理:去噪-输出图像" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/18/SmGpOXbzE2yCd3h.jpg" alt="图像处理:高斯模糊-输出图像" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/18/82LqJy6WCzxMtOY.jpg" alt="图像处理:套红-输出图像" style="width: 30%;">
</div>

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/algorithm/image/img)

## 代码实现

```html
    public static BufferedImage denoiseImage(BufferedImage originalImage) {
        int width = originalImage.getWidth();
        int height = originalImage.getHeight();

        byte[][] imageArray = convertTo2DArray(originalImage);

        int patchSize = 3;
        int searchWindowSize = 11;
        double smoothingParameter = 10.0;
        double distanceThreshold = 30.0;

        IntStream.range(0, height).parallel().forEach(y -> {
            IntStream.range(0, width).parallel().forEach(x -> {
                byte[][] patch = getPatch(imageArray, x, y, patchSize);

                double weightSum = 0.0;
                double result = 0.0;

                int halfSearch = searchWindowSize / 2;

                for (int i = -halfSearch; i <= halfSearch; i++) {
                    for (int j = -halfSearch; j <= halfSearch; j++) {
                        int newX = x + i;
                        int newY = y + j;

                        if (isValidCoordinate(newX, newY, width, height)) {
                            byte[][] similarPatch = getPatch(imageArray, newX, newY, patchSize);

                            double distance = calculateDistance(patch, similarPatch);

                            if (distance > distanceThreshold) {
                                continue;
                            }

                            double weight = Math.exp(-distance / (2.0 * smoothingParameter * smoothingParameter));
                            weightSum += weight;

                            result += weight * getValue(imageArray, newX, newY);
                        }
                    }
                }

                result /= weightSum;

                imageArray[y][x] = (byte) result;
            });
        });

        // 将二维数组转换为BufferedImage
        return convertToBufferedImage(imageArray);
    }

    // 添加一个用于计算距离的辅助方法
    private static double calculateDistance(byte[][] patch1, byte[][] patch2) {
        double sum = 0.0;
        for (int i = 0; i < patch1.length; i++) {
            for (int j = 0; j < patch1[0].length; j++) {
                sum += Math.pow(patch1[i][j] - patch2[i][j], 2);
            }
        }
        return sum / (patch1.length * patch1[0].length);
    }
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
