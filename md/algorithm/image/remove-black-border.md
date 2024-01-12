## 背景

&ensp;&ensp;在人工智能相关的处理中，有种说法是训练数据集对人工智能的性能和效果有着重要的影响。一个高质量的训练数据集可以帮助模型更好地理解和学习任务，从而提高其性能。本文主要针对图像处理，期望对图像的四周黑边进行处理，从而提升训练数据集的质量。

## 概述

&ensp;&ensp;使用JAVA实现Canny边缘检测算法，针对一直输入图像，先进行边缘检测，之后基于边缘检测的结果，完成图像的去黑边操作。

## 实现效果示例

&ensp;&ensp;使用JAVA实现Canny边缘检测算法，针对一直输入图像，先进行边缘检测，之后基于边缘检测的结果，完成图像的去黑边操作。

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/01/12/FydGLQIaxNOE9DY.jpg" alt="图像去黑边-输入图像" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/12/4jB3i2xh9UADycJ.jpg" alt="图像去黑边-Canny边缘检测" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/12/8SC6K59urxUg4HE.jpg" alt="图像去黑边-输出图像" style="width: 30%;">
</div>

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/algorithm/image/img)

## 代码

&ensp;&ensp; [基于JAVA的图像去黑边处理](https://gitee.com/alexgaoyh/pap-base/tree/v1/src/main/java/com/pap/base/util/image/edgedetector)

```java
public class ImageIOUtils {

    /**
     * 传入需要处理的图像绝对路径，完成去黑边操作之后保存至指定路径
     * @throws Exception
     */
    public static void removeBlackEdge() throws Exception {
        BufferedImage bufferedImage = loadImage("input_image_path");
        BufferedImage bufferedImage1 = removeBlackEdge(bufferedImage);
        saveImage(bufferedImage1, "output_image_path", "jpg");
    }
}
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
3. https://github.com/JianQuanMa/CannyEdgeDetector
