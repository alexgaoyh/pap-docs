# [NLP] langchain-ChatGLM 本地知识库

## 介绍

&ensp;&ensp;[langchain-ChatGLM](https://github.com/imClumsyPanda/langchain-ChatGLM)基于本地知识库的问答应用，建立一套对中文场景与开源模型支持友好、可离线运行的知识库问答解决方案。

## 背景

&ensp;&ensp;从2023年初，断断续续的在工作中试用了ChatGPT，惊叹其对工作的帮助，近段时间和朋友聊天，思考能够在某些细分的领域类似CahtGPT的模型，构建私有的本地离线知识库？

&ensp;&ensp;通过资料查询，优先选择 langchain-ChatGLM 进行处理。

## 部署与启动

&ensp;&ensp; 相关环境：Nvidia-RTX-A4000(显存16G)； 预装 Ubuntu20.04, Python 3.9, Pytorch 2.0, CUDA 11.7, cuDNN 8, NVCC, VNC； 

&ensp;&ensp; 按照步骤执行如下命令，下载最新代码，并且进行环境安装。

1. git clone https://github.com/imClumsyPanda/langchain-ChatGLM.git
2. cd langchain-ChatGLM
3. pip3 install -r requirements.txt
4. pip3 install -U gradio
5. pip3 install modelscope

&ensp;&ensp; 模型下载与配置，模型下载完毕后，修改 /configs/model_config.py 文件，将默认的 "text2vec"、"chatglm-6b" 部分的模型路径改为本地路径（避免服务器无法访问 huggingface.co）

1. https://huggingface.co/THUDM/chatglm-6b/tree/main
2. https://huggingface.co/GanymedeNil/text2vec-large-chinese/tree/main

&ensp;&ensp; 启动

1. python webui.py

## 试用

&ensp;&ensp; 作者租用了一台 Nvidia-RTX-A4000(显存16G)的机器（通过监控，模型运行过程中，显存占用在14.25G左右徘徊，硬盘空间占用不到20G）。

&ensp;&ensp; 首先简单试用一下当前模型，其中Bing搜索问答这里并没有进行深究（针对当前需求不作为重点），效果如下图所示，图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/langchain-ChatGLM-simple.png) [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/langchain-ChatGLM-bing.png)

![简单测试.png](https://s2.loli.net/2023/06/29/lB7ayTJ3sLmXVQu.png)

![bing搜索问答.png](https://s2.loli.net/2023/06/29/4t7xylgDKo8zPVc.png)

&ensp;&ensp; 通过试用，简单的对话功能能够实现，至此开始添加私有的知识库，本文选择了人物的生平信息作为知识信息，整理为TXT文件，每一行代表一个人的生平信息，通过上传TXT文件将其加入到知识库中，同时也试用了一下'添加单条内容'的功能，并通过对话得到知识信息，效果如下图所示，图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/langchain-ChatGLM-load-kg.png) [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/langchain-ChatGLM-using-kg.png)

![加载知识库信息.png](https://s2.loli.net/2023/06/29/QOSiGKN256HwaC3.png)

![查询知识库信息.png](https://s2.loli.net/2023/06/29/YNgwqzUpj4P91Td.png)
