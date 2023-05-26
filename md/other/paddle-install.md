1. 安装 Anaconda
    1. 访问 https://repo.anaconda.com/archive/?C=M&O=A 下载符合版本的 Anaconda
   3. 本文在 Ubuntu18 下，下载的是： Anaconda3-5.3.1-Linux-x86_64.sh
    3. 执行 bash Anaconda3-5.3.1-Linux-x86_64.sh 进行安装
    4. 查看许可证部分输入 yes 表示同意
    5. 选择合适的目录，不需要更改的话直接回车即可
    6. 安装完毕后输入 yes 表示确定使用 conda init 来启动
    7. 如果输入 conda 显示找不到命令，则执行 source ~/.bashrc 命令即可
    8. 可选 conda update -n base -c defaults conda 进行升级


2. 安装 CUDA
    1. 由于笔者使用的机器的机器存在GPU，则优先配置一下 CUDA， 注意这里需要确定版本；
    2. 执行 nvidia-smi 命令，根据 cuda的版本，在 https://developer.nvidia.com/cuda-toolkit-archive 链接中选择合适的版本进行处理
    3. https://developer.nvidia.com/cuda-11-6-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=18.04&target_type=runfile_local
    4. 选择 runfile_local 进行命令执行；


3. 安装 cudnn
    1. 在 https://developer.nvidia.com/rdp/cudnn-archive 链接中进行下载，注意与 paddlepaddle 官网的安装进行匹配（对于 CUDA 11.6，需要搭配 cuDNN 8.4.0(多卡环境下 NCCL>=2.7)）
    2. 匹配过后，本文选择了 cudnn-linux-x86_64-8.4.0.27_cuda11.6-archive.tar.xz  版本
    3. tar -xvJf cudnn-linux-x86_64-8.4.0.27_cuda11.6-archive.tar.xz
    4. 详细的安装可进行其他搜索引擎的搜索


4. paddlepaddle 初始化
    1. https://www.paddlepaddle.org.cn/documentation/docs/zh/install/conda/linux-conda.html#anchor-0
    2. 创建 Anaconda 虚拟环境 conda create -n paddle_env python=3.10
    3. 进入 Anaconda 虚拟环境	conda activate paddle_env
    4. 按照官方文档执行其他的环境检查：which python3; python3 --version
    5. 添加清华源， 注意笔者在添加的时候，没有使用 https，而改为 http，原因是在后续使用的时候，会出现ssl握手失败
		
        conda config --add channels http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
		
        conda config --add channels http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
		
        conda config --set show_channel_urls yes
		
	6. 执行安装 对于 CUDA 11.6，需要搭配 cuDNN 8.4.0(多卡环境下 NCCL>=2.7)，安装命令注意这里同样没有使用 https，而改为 http，原因同样是后续出现的ssl握手失败
		
        conda install paddlepaddle-gpu==2.4.2 cudatoolkit=11.6 -c http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/Paddle/ -c conda-forge
		
	7. 安装验证，经过验证安装成功。这里很奇怪的一点是，刚开始验证失败，在经历了一次重启之后，安装验证成功；


5. 安装包版本
    1. Anaconda3-5.3.1-Linux-x86_64.sh
    2. cuda_11.6.0_510.39.01_linux.run
    3. cudnn-linux-x86_64-8.4.0.27_cuda11.6-archive.tar.xz
    4. NVIDIA-Linux-x86_64-525.116.04.run

6. paddleocr
   1. 在实际使用过程中，会发现 python==3.10 版本些许高，所以重新安装了基于 python==3.8 的 paddlepaddle
   2. pip install paddleocr=2.2  这里安装了2.2版本
   3. pip install opencv-python==4.1.2.30 重新更改 opencv-python 的版本，解决 module cv2 has no attribute gapi_wip_gst_gstreamerpipeline
   4. pip install numpy=1.23.1 重新更改 numpy 的版本，解决 module numpy has no attribute bool 
   5. 执行 paddleocr --image_dir 11.jpg --use_angle_cls true 命令的时候，会遇到GPU环境下的乱码， 可以在最后部分增加  --use_gpu false
   6. 如下是具体的执行命令，相比于官方的命令，移除了 result = result[0]的部分，并且修改了 font_path 的字体路径，至此官方示例已完成。
```shell
   from paddleocr import PaddleOCR,draw_ocr
   ocr = PaddleOCR(use_angle_cls=True, lang='ch', use_gpu=False)
   img_path = '/home/alex/paddleocr/11.jpg'
   result = ocr.ocr(img_path, cls=True)
   for idx in range(len(result)):
   res = result[idx]
   for line in res:
   print(line)
   
   # draw result
   from PIL import Image
   # result = result[0]
   image = Image.open(img_path).convert('RGB')
   boxes = [line[0] for line in result]
   txts = [line[1][0] for line in result]
   scores = [line[1][1] for line in result]
   im_show = draw_ocr(image, boxes, txts, scores, font_path='/usr/share/fonts/truetype/arphic/uming.ttc')
   im_show = Image.fromarray(im_show)
   im_show.save('/home/alex/result.jpg')
```
