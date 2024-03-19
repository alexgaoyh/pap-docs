# [人脸修复]基于CodeFormer的人脸修复模型配置

## 背景

&ensp;&ensp;近期在进行图像处理相关需求的时候，需要对旧照片进行修复，并了解到了 CodeFormer 这个基于AI技术深度学习的人脸复原模型，对其进行环境配置和试用，特此记录。

## 步骤

1. 源码下载并解压进入目录		
   1. git clone https://github.com/sczhou/CodeFormer.git
   2. unzip CodeFormer-master.zip
   3. cd CodeFormer-master
2. 环境安装(清华源有时候会慢,在安装过程中使用的阿里云的源)：
   1. conda create -n codeformer python=3.8 -y
   2. conda activate codeformer
   3. pip3 install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple
   4. python basicsr/setup.py develop
   5. conda install -c conda-forge dlib
3. 模型下载后放置到指定文件夹
   1. weights/facelib	文件夹存放 
      1. yolov5n-face.pth
      2. detection_mobilenet0.25_Final.pth
      3. detection_Resnet50_Final.pth
      4. parsing_parsenet.pth
      5. yolov5l-face.pth
   2. weights/CodeFormer 文件夹存放
      1. codeformer.pth
4. 测试模型
   1. 单独图片人脸修复：python inference_codeformer.py -w 0.2 --has_aligned --input_path face.jpg
      1. 输出： Background upsampling: False, Face upsampling: False [1/1] Processing: face.jpg All results are saved in results/test_img_0.2


## 执行过程
<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/19/FSUlInOHNTRPkyK.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [人脸修复]基于CodeFormer的人脸修复模型配置-修复效果
    </div>
</center>

## 总结

&ensp;&ensp;基于CodeFormer，搭建环境后对人脸进行修复，并举出修复效果。

## 参考

1. https://gitee.com/alexgaoyh
2. https://pap-docs.pap.net.cn/
