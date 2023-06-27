# [Paddle] ERNIE-UIE 通用信息抽取模型（含自定义细分领域模型训练） 

## 介绍

&ensp;&ensp;[ERNIE-UIE](https://aistudio.baidu.com/aistudio/modelsdetail?modelId=22)信息抽取模型可以进行关键信息抽取，可参照官网安装流程进行配置和使用。

&ensp;&ensp;但是在实际的细分领域中（细分的应用场景），信息抽取的效果并不好（中文书写习惯截然不同），本文按照官网的方式，进行模型训练从而进一步提升效果，并进行记录。

## 环境配置

1. <h4>Paddle、ERNIE-UIE</h4>
   1. [Paddle Install](https://www.paddlepaddle.org.cn/install/quick?docurl=/documentation/docs/zh/install/pip/linux-pip.html)
   2. [ERNIE-UIE Install](https://aistudio.baidu.com/aistudio/modelsdetail?modelId=22)
   3. 正确安装后能够正确返回信息抽取的结果；

2. <h4>doccano数据标注</h4>
   1. [介绍](https://github.com/PaddlePaddle/PaddleNLP/blob/develop/model_zoo/uie/doccano.md)
   2. [安装](https://github.com/doccano/doccano)
   3. doccano环境安装成功后，登录系统并创建一个[序列标注]类型的项目[regex]，如图所示定义了三个Tag: ['start', 'label', 'end']。图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/uie-doccano-1.png)
      ![创建一个序列标注项目](https://s2.loli.net/2023/06/27/YZWz1AaglKcwXo2.png)
   4. 在[regex]项目下，导入数据集并且进行标注，如图所示，每一段话按顺序标注['start', 'label', 'end'] 三个部分。图片无法正常查看的话， [访问](https://gitee.com/alexgaoyh/pap-docs/blob/master/md/other/img/uie-doccano-2.png)
      ![使用doccano进行数据标注](https://s2.loli.net/2023/06/27/lixSQIkemVDEPFn.png)
   5. 如上图所示，选中数据进行导出，会下载一个zip文件夹，内部包含一个名为[admin.jsonl]的文件，将其重命名为 doccano_ext.json。


3. <h4>PaddleNLP</h4>
   1. 使用 git clone 命令下载[PaddleNLP](https://github.com/PaddlePaddle/PaddleNLP/)。
   2. 进入到 /model_zoo/uie 文件夹并创建 data 文件夹，并将上传上述生成的 doccano_ext.json 文件。
   3. 进行数据转换，执行如下命令，会在 data 文件夹下生成：train.txt、test.txt、dev.txt、sample_index.json 这些文件。

        ```shell
        python doccano.py  --doccano_file ./data/doccano_ext.json  --task_type "ext"  --save_dir ./data  --negative_ratio 5
        ```
      
   4. 建议在GPU环境下进行[模型微调](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/model_zoo/uie#43-%E6%A8%A1%E5%9E%8B%E5%BE%AE%E8%B0%83)，作者租用了一台 A30(24G显存)进行的训练。
   5. 使用定制的模型进行预测，修改 ERNIE-UIE 官网提供的代码（注意 Taskflow 是通过task_path指定模型权重文件的路径）
   
        ```shell
        schema = ['start', 'label', 'end']
        my_ie = Taskflow("information_extraction", schema=schema, task_path='./checkpoint/model_best')
        ```
   6. 通过结果能改看到模型发生了变化。

        ```html
        [{
          'end': [{
            'end': 24,
            'probability': 0.8684210982049407,
            'start': 21,
            'text': '第三章'
          }],
          'label': [{
            'end': 12,
            'probability': 0.9925105558749578,
            'start': 10,
            'text': '借阅'
          }],
          'start': [{
            'end': 3,
            'probability': 0.8233676770565523,
            'start': 0,
            'text': '第二章'
          }]
        }]
        ```
      
4. <h4>总结</h4>

&ensp;&ensp;通过标注少量数据对 UIE 模型进行微调，将其应用到垂直细分领域，提升了信息提取的效果，能够更方便的将其应用到细分的实际应用场景中。

5. <h4>参考</h4>
   1. https://github.com/PaddlePaddle/PaddleNLP
   2. https://github.com/doccano/doccano
   3. https://pap-docs.pap.net.cn/#/md/other/paddle/paddle-install
   4. 
