# [Paddle] ERNIE-Layout 使用测试 - 文心多语言跨模态布局增强文档智能大模型

## 介绍

&ensp;&ensp;近期遇到需要从电子文档中进行内容提取的需求，突然想到 Paddle 的 ERNIE-Layout 模型，对其进行分析和测试。

&ensp;&ensp;采用此方法，避免了很笨的 OCR + 正则匹配 的思路。

## 应用场景

1. 电子文档
    1. 对表格类的文档进行内容提取（发票、票据、简历）
    2. 对文档进行问答

## 使用方法

1. 安装：
   1. [paddlepaddle环境安装](https://www.paddlepaddle.org.cn/install/quick)
      1. 如果是GPU环境的话，执行 nvidia-smi、nvcc -V 两个命令，根据结果进行不同命令的安装；
      2. 如果是CPU环境的话，可直接进行安装；
      3. 执行如下命令： pip install --upgrade paddleocr  pip install --upgrade paddlenlp
      4. [官方示例代码](https://aistudio.baidu.com/aistudio/modelsdetail?modelId=23)，执行官网示例代码验证环境安装是否成功；图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/ERNIE-Layout-offical-example.png)
         ![从发票图片中提取发票号码、校验码](https://s2.loli.net/2023/06/26/i5KLv4TzHsWwc7Z.png)
      5. 针对具体业务图片进行信息提取测试(部分字段被隐藏，但是不影响去理解当前模型可以针对此格式的图片进行信息提取)，使用非官方图片，图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/ERNIE-Layout-customer-example-question.png)![非官方测试图片](https://s2.loli.net/2023/06/26/3a1D5UJPmQXwOI4.png)
      6. 测试结果，图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/ERNIE-Layout-customer-example-answer.png)![非官方测试图片结果](https://s2.loli.net/2023/06/26/Kqcm2Xg5okxQeEJ.png)
2. 应用
   1. 可以外面包一层 django 框架进行服务化；
   2. https://github.com/PaddlePaddle/FastDeploy 当前未看到关于 ERNIE-Layout 部分的介绍；

## 问题解决

1. ImportError: cannot import name '_registerMatType' from 'cv2.cv2'
   1. https://github.com/opencv/opencv-python/issues/591
   2. 

        ```shell
        pip install --upgrade opencv-python
        pip install --upgrade opencv-contrib-python
        pip install --upgrade opencv-python-headless
        ```

2. ERROR: After October 2020 you may experience errors when installing or updating packages. This is because pip will change the way that it resolves dependency conflicts.
We recommend you use --use-feature=2020-resolver to test your packages with the new resolver before it becomes the default.
   1. 本文在安装和测试的过程中，忽略了此问题；
   2. 根据官方介绍，可以在 pip install 命令后添加 --use-feature=2020-resolver 去解决；

## 相关参考

1. https://github.com/opencv/opencv-python/issues/591
2. https://www.matpool.com/  注册过程中可用邀请码： r6LeEKPs7ivbaLy
3. https://developer.nvidia.com/cuda-11-7-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=runfile_local
4. https://www.paddlepaddle.org.cn/
5. https://aistudio.baidu.com/aistudio/modelsdetail?modelId=23
