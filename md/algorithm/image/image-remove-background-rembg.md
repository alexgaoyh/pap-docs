# [图像处理]基于Rembg的图像背景自动去除工具

## 背景

&ensp;&ensp;近期在进行图像处理相关需求的时候，处理了图像水印相关需求，想到了图像抠图，去除图像背景，通过查询使用 Rembg 开源库，对其进行环境配置和试用，特此记录。

## 步骤

1. 安装
   1. 命令行安装 命令行安装 pip install rembg
   2. 源码安装 
      1. git clone https://github.com/danielgatis/rembg.git
      2. pip install .
2. 额外依赖
   1. pip install click
   2. pip install filetype
   3. pip install watchdog
   4. pip install aiohttp
   5. pip install gradio
   6. pip install asyncer 
3. 复制模型文件
   1. cp u2net.onnx /root/.u2net/
4. 添加环境变量
   1. vi ~/.bashrc
   2. export OMP_NUM_THREADS=10
   3. . ~/.bashrc
5. 执行命令
   1. rembg i after.png after-out.png

## 执行过程
<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/19/p45Yku9KZAca8is.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [图像处理]基于Rembg的图像背景自动去除工具-输入图像1
    </div>
</center>

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/19/Uk63SDtgZORdyLl.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [图像处理]基于Rembg的图像背景自动去除工具-输出图像1
    </div>
</center>

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/19/M5mfs4jneCDxQRH.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [图像处理]基于Rembg的图像背景自动去除工具-输入图像2
    </div>
</center>

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"  src="https://s2.loli.net/2024/03/19/boVBF1tTzfsqcpm.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        [图像处理]基于Rembg的图像背景自动去除工具-输出图像2
    </div>
</center>

## 总结

&ensp;&ensp;基于 Rembg 图像背景自动去除工具，搭建环境后对图片进行抠图。

## 参考

1. https://gitee.com/alexgaoyh
2. https://pap-docs.pap.net.cn/
