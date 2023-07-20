# [Huggingface]系列文章(1)-认识Transformers

## 背景

&ensp;&ensp;本文是[Huggingface]系列文章的第一篇，期望通过如下的介绍，向用户展示[Huggingface]可以做到哪些事情。

## 环境安装 

&ensp;&ensp;分为三个代码段落，首先使用conda初始化python环境，其次安装 transformers，最后使用sentiment-analysis的文本分类任务。

```shell
conda create -n huggingface python=3.8
conda activate huggingface
```

```shell
pip install transformers -i https://pypi.tuna.tsinghua.edu.cn/simple

pip install tensorflow -i https://pypi.tuna.tsinghua.edu.cn/simple

pip install torch -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 模型测试

&ensp;&ensp;由于国内网络环境的限制，作者在实际执行过程中，经常出现超时情况，可以将模型手动下载后进行处理。

### 文本分类

&ensp;&ensp;如上所述网络问题，故附上[模型下载](https://huggingface.co/distilbert-base-uncased-finetuned-sst-2-english/tree/main)，将模型下载至本地后指定模型路径，如下所示。

```shell
mkdir huggingface
cd huggingface
vim sentiment-analysis.py

    from transformers import pipeline
    classifier = pipeline("sentiment-analysis", model="/home/alex/huggingface/model/distilbert-base-uncased-finetuned-sst-2-english")
    result = classifier("I've been waiting for a HUggingface course my whole life.")
    print(result)
    
    
python sentiment-analysis.py
```

执行结果如下图所示，如图片无法正常查看的话， [点击此处访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/hugging-sentiment-anaylis-result.png)

![hugging-sentiment-anaylis-result.png](https://s2.loli.net/2023/07/20/x7RJw6PM3pUylZX.png)

### 文本生成

&ensp;&ensp;针对文本生成任务，同样可以采用如上的方式，[模型下载](https://huggingface.co/bigscience/bloomz-1b1/tree/main)

```shell
from transformers import pipeline

generate = pipeline("text-generation", model="/home/alex/huggingface/model/bloomz-1b1")
result = generate("我的英文名是alexgaoyh，我喜欢打篮球，", max_new_tokens=100)
print(result)
```
执行结果如下图所示，如图片无法正常查看的话， [点击此处访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/hugging-text-genetarot-result.png)

![hugging-text-genetarot-result.png](https://s2.loli.net/2023/07/20/rgtPIqHwfGpNia4.png)

## 常见问题

&ensp;&ensp;在环境安装过程中，发现部分文章只介绍了安装 transformers，对其可能需要的 tensorflow、PyTorch 并没有提及。

&ensp;&ensp;在运行代码示例的过程中，部分文章介绍会自动进行模型下载，但是受国内网络环境的影响，并不能很好的进行模型下载，故本文提前将模型下载完毕，并指定位置。

&ensp;&ensp;pipeline中可以显示指定其他的模型进行使用，而不一定非要是默认的模型，可以在 HuggingFace 的模型库中进行下载。

## 参考

1. https://huggingface.co/docs/transformers/v4.27.2/zh/quicktour
2. https://zhuanlan.zhihu.com/p/448852278
3. https://gitee.com/alexgaoyh/pap-docs


