## 背景

&ensp;&ensp;在人工智能相关的处理中，有种说法是训练数据集对人工智能的性能和效果有着重要的影响。一个高质量的训练数据集可以帮助模型更好地理解和学习任务，从而提高其性能。本文主要针对图像处理，期望对图像进行纠偏处理，从而提升训练数据集的质量。

## 概述

&ensp;&ensp;使用JAVA实现傅里叶频谱平移图，之后使用霍夫变化获得图像的倾斜角度，最后进行纠偏。

## 实现效果示例

&ensp;&ensp;首先对原始图像进行缩放，图像大小调整至2的幂次方，之后进行傅里叶频谱变化，基于频谱图使用霍夫变换获得倾斜角度，最终完成纠偏操作。

<div style="display: flex; justify-content: space-between;">
    <img src="https://s2.loli.net/2024/01/12/woFjHpcBnLJxb3s.jpg" alt="图像自动纠偏-输入图像" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/12/RT91ju5gqJPo8ev.jpg" alt="图像自动纠偏-傅里叶频谱" style="width: 30%;">
    <img src="https://s2.loli.net/2024/01/12/a5MKB2mLwYuUhFT.jpg" alt="图像自动纠偏-霍夫变换纠偏" style="width: 30%;">
</div>

&ensp;&ensp;图像备份: [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/algorithm/image/img)

## 代码

&ensp;&ensp; [基于JAVA的图像自动纠偏处理](https://gitee.com/alexgaoyh/pap-base/tree/v1/src/main/java/com/pap/base/util/image)

```java
public class ImageIOUtils {

    /**
     * 传入需要处理的图像绝对路径，完成自动纠偏操作之后保存至指定路径
     * @throws Exception
     */
    public static void removeBlackEdge() throws Exception {
        BufferedImage bufferedImage = loadImage("input_image_path");
        BufferedImage bufferedImage1 = autoCorrection(bufferedImage, "tmp_image_path");
        saveImage(bufferedImage1, "output_image_path", "jpg");
    }
}
```

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
2. https://github.com/NickBodliev/HoughLines
