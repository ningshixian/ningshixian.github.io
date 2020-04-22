---
layout:     post
title:      bert_as_service使用指南
subtitle:   手把手教你部署bert_as_service服务
date:       2019-11-08
author:     NSX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 技术
    - 教程
---


# bert-as-service服务

[Github地址](https://github.com/hanxiao/bert-as-service)

> Using BERT model as a sentence encoding service, i.e. mapping a variable-length sentence to a fixed-length vector.

![](https://github.com/hanxiao/bert-as-service/blob/master/.github/demo.gif?raw=true)


### bert_as_service 是什么

BERT是Google为预训练语言表示而开发的NLP模型。它利用了在网络上公开提供的大量纯文本数据，并且以无人监督的方式进行了培训。
对于每种语言，预训练BERT模型是一个相当昂贵但一次性的过程。幸运的是，Google发布了一些经过预先训练的模型，[您可以从此处下载](https://github.com/google-research/bert#pre-trained-models)。

`bert-as-service` 使用BERT作为句子编码器，并通过ZeroMQ将其托管为服务，从而使您可以仅用两行代码将句子映射为定长表示形式



### bert_as_service 特点

- 🔭 最新技术：以Google AI发布的经过预训练的12/24层BERT模型为基础，这是NLP社区中的一个里程碑。
- 🐣 易于使用：只需两行代码即可获得句子/令牌级别的编码。
- ⚡️ 快速：单个Tesla M40 24GB上每秒900句。低延迟，针对速度进行了优化。参见基准。
- 🐙 可扩展：可在多个GPU和多个客户端上顺利流畅地扩展，而无需担心并发性。参见基准。
- 💎 可靠：经过数十亿个句子的测试；连续几天运行，没有间断或OOM或任何令人讨厌的异常。



### bert_as_service 安装

```
source activate tensorflow  # 开启虚拟环境
pip install bert-serving-server==1.9.6 # 服务端
pip install bert-serving-client==1.9.6  # 客户端，与服务端互相独立
pip install tensorflow>=1.13.1
```

注意，服务器必须在运行的Python> = 3.5与Tensorflow> = 1.10（单点-10）。同样，该服务器不支持Python 2！

☝️出于以下考虑，客户端可以同时在Python 2和3上运行。



### bert_as_service 入门

1. 下载预训练的BERT模型

   下载下面列出的模型，然后将zip文件解压缩到某个文件夹，例如 `/tmp/chinese_L-12_H-768_A-12/`

| 模型 | 说明 |
| -- | -- |
| BERT-Base, Uncased	| 12-layer, 768-hidden, 12-heads, 110M parameters |
| BERT-Large, Uncased	| 24-layer, 1024-hidden, 16-heads, 340M parameters |
| [BERT-Base, Cased](https://storage.googleapis.com/bert_models/2018_10_18/cased_L-12_H-768_A-12.zip)	| 12-layer, 768-hidden, 12-heads , 110M parameters |
| [BERT-Large, Cased](https://storage.googleapis.com/bert_models/2018_10_18/cased_L-24_H-1024_A-16.zip)	| 24-layer, 1024-hidden, 16-heads, 340M parameters |
| BERT-Base, Multilingual Cased (New)	| 104 languages, 12-layer, 768-hidden, 12-heads, 110M parameters |
| BERT-Base, Multilingual Cased (Old)	| 102 languages, 12-layer, 768-hidden, 12-heads, 110M parameters |
| [BERT-Base, Chinese](https://storage.googleapis.com/bert_models/2018_11_03/chinese_L-12_H-768_A-12.zip)	| Chinese Simplified and Traditional, 12-layer, 768-hidden, 12-heads, 110M parameters |

2. 开启BERT服务

   安装服务器后，可以使用 `bert-serving-start` 命令启动服务

   `bert-serving-start -model_dir /data01/dai/jupyter/bert/chinese_L-12_H-768_A-12/ -num_worker=4 -port=5555 -port_out=5556 -cpu`

   其中，`-model_dir` 是预训练模型的路径，`-num_worker` 是线程数，表示同时可以处理多少个并发请求

![](https://github.com/hanxiao/bert-as-service/blob/master/.github/server-demo.gif?raw=true)

​	**或者, 使用脚本`bert_server.py`启动服务**

​	`nohup python -u bert_server.py > log.txt 2>&1 &`

​	bert_server.py 例子：
```
from bert_serving.server.helper import get_args_parser
from bert_serving.server import BertServer
args = get_args_parser().parse_args(['-model_dir', '/data01/dai/jupyter/bert/chinese_L-12_H-768_A-12',
                                     '-port', '5555',
                                     '-port_out', '5556',
                                     '-max_seq_len', '50',
                                     '-num_worker','3',
                                     '-mask_cls_sep',
                                     '-cpu'])
server = BertServer(args)
server.start()
```

3. 在客户端获取句向量（远程使用BERT服务）

   在一台（GPU）机器上启动该服务，然后从另一台（CPU）机器调用该服务，如下所示：

```
from bert_serving.client import BertClient

# 要带上参数 check_token_info=False，否则可能报错
bc = BertClient(ip='10.231.135.107', port=5555, port_out=5556, timeout=60000, check_version=False, check_token_info=False)
bc.encode(['First do it', 'then do it right', 'then do it better'])  #直接输入整个句子不需要提前分词
```

​	它将返回一个ndarray（或List[List[float]]如果您愿意的话），其中每一行都是代表一个句子的固定长度向量。

​	请注意，仅`pip install -U bert-serving-client`在这种情况下才需要服务器端。您也可以通过[HTTP请求调用服务](https://github.com/hanxiao/bert-as-service#using-bert-as-service-to-serve-http-requests-in-json)。



### 想了解更多？

查看我们的教程：

- [在3分钟内构建一个质量检查语义搜索引擎](https://github.com/hanxiao/bert-as-service#building-a-qa-semantic-search-engine-in-3-minutes)
- [服务于微调的BERT模型](https://github.com/hanxiao/bert-as-service#serving-a-fine-tuned-bert-model)
- 获得类似ELMo的上下文词嵌入
- [使用自己的令牌生成器](https://github.com/hanxiao/bert-as-service#using-your-own-tokenizer)
- BertClient与tf.dataAPI一起使用
- 使用BERT功能和tf.estimator API训练文本分类器
- 保存和加载TFRecord数据
- [异步编码](https://github.com/hanxiao/bert-as-service#asynchronous-encoding)
- 广播到多个客户端
- 在仪表板中监控服务状态
- [使用bert-as-service服务于JSON的HTTP请求](https://github.com/hanxiao/bert-as-service#using-bert-as-service-to-serve-http-requests-in-json)
- [BertServer从Python 开始](https://github.com/hanxiao/bert-as-service#starting-bertserver-from-python)



# 参考

[[译] 通过优化 Gunicorn 配置提高性能](https://juejin.im/post/5ce8cab8e51d4577523f22f8)

[https://github.com/hanxiao/bert-as-service#what-is-it](https://github.com/hanxiao/bert-as-service#what-is-it)

[bert-as-service 最新API文档](http://bert-as-service.readthedocs.io/)
