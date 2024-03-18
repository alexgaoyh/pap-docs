# [目标检测]基于YOLOV8的自定义数据集实现水印检测

## 背景

&ensp;&ensp;近期在进行图像处理相关需求的时候，编写了[图像处理-Java-OpenCV-水印编码/解码](https://pap-docs.pap.net.cn/#/md/algorithm/image/image-opencv-dct-watermark)这篇文章，将水印隐藏到了图像中，关于版权保护这里，突然想到了之前接触到的一个检查CMS系统中是否使用了包含水印的图像，故编写此文。

## 思路

&ensp;&ensp;使用yolo，自定义数据集之后划分数据集，灌入模型进行训练，并进行测试得到结果。

## 步骤

1. 从GitHub上下载源码		https://github.com/ultralytics/ultralytics/tree/v8.1.0
2. 使用 conda 创建环境		conda create --name ultralytics-env python=3.8 -y
3. 激活环境		conda activate ultralytics-env
4. 切换至源码解压路径		cd ultralytics-8.1.0
5. 环境安装(显示Successfully built ultralytics)		pip install -e .
6. (可选)安装数据标注(Successfully built labelImg)		pip install labelImg
7. (可选)可视化界面进行数据标注		labelImg
8. 参照[制作自己的数据集并训练的YOLOv8模型](https://blog.csdn.net/liang_baikai/article/details/132082292)

## 执行过程

&ensp;&ensp;本文使用代码详见： [基于YOLOV8的自定义数据集实现水印检测](https://gitee.com/alexgaoyh/ultralytics/tree/pap-watermark/)

1. 使用 labelImg 创建自定义的数据集，注意左侧选择 yolo
2. 将自定义的数据集拷贝至 datasets/data/images 文件夹，使用 Process.py 切分数据集
3. 请验证 datasets/data/train、datasets/data/test、datasets/data/val 三个文件夹下是否生成对应的数据
4. 执行 train.py 进行训练，请注意 imgsz 参数，如果图像很大，此参数建议同步进行增大
5. 执行 trest.py 验证模型，请对照 test.jpg 和 result.jpg 两张图像

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/18/2pFkOruGnUfIJ9N.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [目标检测]基于YOLOV8的自定义数据集实现水印检测-YoloV8-从源码安装
    </div>
</center>

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/18/GSlays4kopxTnzV.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [目标检测]基于YOLOV8的自定义数据集实现水印检测-YoloV8-labelImg-标注自定义数据集
    </div>
</center>

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/18/CyO68tNdUMlAqLe.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [目标检测]基于YOLOV8的自定义数据集实现水印检测-YoloV8-切分数据集
    </div>
</center>

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/18/Ijry4BLh9fXdOpK.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [目标检测]基于YOLOV8的自定义数据集实现水印检测-YoloV8-使用自定义数据集训练
    </div>
</center>

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/18/ZJ54NgC1a3Y8vdA.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [目标检测]基于YOLOV8的自定义数据集实现水印检测-YoloV8-验证模型
    </div>
</center>

## 总结

&ensp;&ensp;基于Yolo，使用自定义数据集，标注数据集中的水印区域，训练模型后进行测试，得到标注结果。本文使用源码详见[基于YOLOV8的自定义数据集实现水印检测](https://gitee.com/alexgaoyh/ultralytics/tree/pap-watermark/)

## 参考

1. https://gitee.com/alexgaoyh/ultralytics/tree/pap-watermark/
2. https://github.com/ultralytics/ultralytics/tree/v8.1.0
3. https://blog.csdn.net/liang_baikai/article/details/132082292
