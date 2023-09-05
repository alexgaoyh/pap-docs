# [PaddleGAN]人脸表情迁移-视频换脸

## 背景

&ensp;&ensp;最近和朋友聊天，突然聊到了视频编辑的换脸功能，对此功能进行了调研，通过分析，最终选择Paddle飞浆的PaddleGAN的 "First Order Motion" 进行视频换脸功能。

## 环境配置

&ensp;&ensp;废话不多说，直接写出来对应的命令，如下所示

```shell
python -m pip install paddlepaddle-gpu==2.1.3.post112 -f https://www.paddlepaddle.org.cn/whl/linux/mkl/avx/stable.html

pip install opencv_python-4.6.0.66-cp36-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl 

python3 -m pip install --upgrade ppgan==2.0.0

unzip PaddleGAN-release-2.0.zip

cd PaddleGAN-release-2.0/applications/

python -u tools/first-order-demo.py       --driving_video ../docs/imgs/fom_dv.mp4      --source_image ../docs/imgs/fom_source_image.png      --ratio 0.4      --relative --adapt_scale

pip install 'protobuf~=3.19.0'

python -u tools/first-order-demo.py       --driving_video ../docs/imgs/fom_dv.mp4      --source_image ../docs/imgs/fom_source_image.png      --ratio 0.4      --relative --adapt_scale

```

### 环境配置解释

#### GPU环境

&ensp;&ensp;本人租用了一台云服务器：GPU NVIDIA RTX A2000 显卡内存12 GB，预装：Python 3.7, CUDA 11.2, cuDNN 8.0.5, Ubuntu 18.04, VNC, Numba

#### PaddleGAN

&ensp;&ensp;最开始选择 ppgan 的最新2.1版本，但是遇到了 [__init__() got an unexpected keyword argument 'slice_size'](https://github.com/PaddlePaddle/PaddleGAN/issues/788) 问题，所以最终切换到 ppgan==2.0.0 版本。

#### PaddlePaddle

&ensp;&ensp;根据PaddleGAN官方的[安装文档](https://github.com/PaddlePaddle/PaddleGAN/blob/release/2.0/docs/zh_CN/install.md)，环境依赖中 PaddlePaddle >= 2.1.0，这里直接选择最低的 2.1.0 版本，
并根据PaddlePaddle的[官方安装文档](https://www.paddlepaddle.org.cn/install/old?docurl=/documentation/docs/zh/install/pip/linux-pip.html#old-version-anchor-33-Linux%20%E5%AE%89%E8%A3%85)进行安装，注意这里
需要先查看GPU的环境配置，执行命令 nvidia-smi 后选择合适的 CUDA 版本对应的 PaddlePaddle 安装命令。

#### opencv_python

&ensp;&ensp;本人在安装PaddleGAN的过程中，发现 opencv-python 执行时间会很长，所以直接在[清华镜像](https://pypi.tuna.tsinghua.edu.cn/simple/opencv-python/)选择对应的 whl 进行单独下载安装。

#### Downgrade the protobuf package to 3.20.x or lower.

&ensp;&ensp;在执行 tools/first-order-demo.py 命令的过程中，会提示 protobuf 的版本问题，所以对 protobuf 进行版本降级，命令如上所示。

### 截图

1、 云服务器GPU环境nvidia-smi，图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/ppgan/ppgan-nvidia-smi.png)
![ppgan-nvidia-smi.png](https://s2.loli.net/2023/09/05/hBaNHpPlOARiw1d.png)

2、 protobuf 错误提示，[访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/ppgan/ppgan-protobuf-error.png)
![ppgan-protobuf-error.png](https://s2.loli.net/2023/09/05/xr2l6R1Stydhu3f.png)

3、 first-order-demo.py 执行，[访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/ppgan/ppgan-run.png)
![ppgan-run.png](https://s2.loli.net/2023/09/05/HpkR1Zg9qBvYEre.png)

## 总结

&ensp;&ensp;命令执行完毕后，视频文件将存储至 /PaddleGAN-release-2.0/applications/output/result.mp4 ，可以对生成的视频进行查看。

## 参考链接

1. https://github.com/PaddlePaddle/PaddleGAN/issues/788
2. https://github.com/PaddlePaddle/PaddleGAN/blob/release/2.0/docs/zh_CN/tutorials/motion_driving.md
3. https://www.paddlepaddle.org.cn/install/old?docurl=/documentation/docs/zh/install/pip/linux-pip.html#old-version-anchor-33-Linux%20%E5%AE%89%E8%A3%85
4. https://pap-docs.pap.net.cn/
