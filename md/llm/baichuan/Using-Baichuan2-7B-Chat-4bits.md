## 概述

&ensp;&ensp;《LLM-结合三元组SPO和提示工程来试用Baichuan2-7B-Chat-4bits模型》近期对LLM进行了一些应用场景的思考，其中很简单的一个场景是客服，假设目前所有的知识信息都在一个Excel文档中，首先将其转换为三元组关系，然后结合提示工程技术向LLM进行提问，期望得到反馈。

## 效果

&ensp;&ensp;最左侧是一个Excel表格，包含商品信息，中间的文字部分是将Excel中的数据转换为三元组SPO信息，并且添加上如图所示的提示工程，右侧是模型返回的结果，可以看到能够按照要求返回数据。

![Baichuan2-7B-Chat-4bits-SPO-chat-result.jpg](https://s2.loli.net/2024/05/11/hEC3cNqzTkw4S6u.jpg)

## 调用

&ensp;&ensp;在安装Baichuan2-7B-Chat-4bits后，使用如下代码进行调用，得到返回结果。

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from transformers.generation.utils import GenerationConfig
import os

# 获取当前文件所在的目录路径
current_dir = os.path.dirname(os.path.abspath(__file__))
# 将当前目录和'model'连接起来，获得'model'文件夹的完整路径
model_path = os.path.join(current_dir, 'model/Baichuan2-7B-Chat-4bits')
print('model_path=', model_path)

if __name__ == '__main__':
tokenizer = AutoTokenizer.from_pretrained(model_path, use_fast=False, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(model_path, device_map="auto", torch_dtype=torch.bfloat16, trust_remote_code=True)
model.generation_config = GenerationConfig.from_pretrained(model_path)
messages = []
messages.append({"role": "user", "content": "解释一下“温故而知新”"})
response = model.chat(tokenizer, messages)
print(response)
```

## 部署

```shell
git clone https://github.com/baichuan-inc/Baichuan2.git

cd Baichuan2

pip install -r requirements.txt

## 处理安装包的兼容问题，Baichuan2-7B-Chat-4bits
pip install bitsandbytes==0.41.1
pip install accelerate-0.25.0

## 同样的可以将如上的python脚本放到 ${Baichuan2} 文件夹下。
## 匹配如上的python脚本，需要下载模型文件到 ${Baichuan2}/model/Baichuan2-7B-Chat-4bits 文件夹下。
```

## 总结

&ensp;&ensp;百川的Baichuan2-7B-Chat-4bits量化模型，在实际部署的时候，显存占用10G左右，略高于其他人的实验结果，对消费级显卡也有一定要求。

&ensp;&ensp;前期之所以选择Baichuan2-7B-Chat-4bits量化模型，其实是想尽可能降低对硬件环境的要求，实际部署的过程中，硬件要求会比预期的高。

&ensp;&ensp;实践过程中，暂未选择私有知识库的形式，也未做出对比，后续会进一步进行对比实现。


## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
