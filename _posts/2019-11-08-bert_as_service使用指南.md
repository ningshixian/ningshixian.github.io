---
layout:     post
title:      bert-as-service使用指南
subtitle:   手把手教你部署bert-as-service服务
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

![img](https://github.com/hanxiao/bert-as-service/raw/master/.github/demo.gif?raw=true)


## bert_as_service 是什么

BERT是Google为预训练语言表示而开发的NLP模型。它利用了在网络上公开提供的大量纯文本数据，并且以无人监督的方式进行了培训。
对于每种语言，预训练BERT模型是一个相当昂贵但一次性的过程。幸运的是，Google发布了一些经过预先训练的模型，[您可以从此处下载](https://github.com/google-research/bert#pre-trained-models)。

`bert-as-service` 使用BERT作为句子编码器，并通过ZeroMQ将其托管为服务，从而使您可以仅用两行代码将句子映射为定长表示形式

<!-- more -->

## bert_as_service 特点

- 🔭 最新技术：以Google AI发布的经过预训练的12/24层BERT模型为基础，这是NLP社区中的一个里程碑。
- 🐣 易于使用：只需两行代码即可获得句子/令牌级别的编码。
- ⚡️ 快速：单个Tesla M40 24GB上每秒900句。低延迟，针对速度进行了优化。参见基准。
- 🐙 可扩展：可在多个GPU和多个客户端上顺利流畅地扩展，而无需担心并发性。参见基准。
- 💎 可靠：经过数十亿个句子的测试；连续几天运行，没有间断或OOM或任何令人讨厌的异常。



# 开始使用📖 

## bert_as_service 安装

```
pip install bert-serving-server==1.10.0 # 服务端
pip install bert-serving-client==1.10.0  # 客户端，与服务端互相独立
pip install tensorflow==1.15.0	(>=1.13.1)
```



## bert_as_service 入门

### 1. 下载预训练模型

前往https://github.com/google-research/bert#pre-trained-models选择模型（本文选择中文模型）下载并解压.

### 2. 开启BERT服务

安装服务器后，可以使用 `bert-serving-start` 命令启动服务

```
bert-serving-start -model_dir /tmp/english_L-12_H-768_A-12/ -num_worker=4 
```

其中，`-model_dir` 是预训练模型的路径，`-num_worker` 是线程数，表示同时可以处理多少个并发请求



**或者, 使用脚本`bert_server.py`启动服务**

```
from bert_serving.server.helper import get_args_parser
from bert_serving.server import BertServer
args = get_args_parser().parse_args(['-model_dir', '.../bert/chinese_L-12_H-768_A-12',
                                     '-port', '5555',
                                     '-port_out', '5556',
                                     '-max_seq_len', '50',
                                     '-num_worker','3',
                                     '-mask_cls_sep',
                                     '-cpu'])
server = BertServer(args)
server.start()
```

### 3. 在客户端获取句向量

> **远程使用BERT服务**

在一台（GPU）机器上启动该服务，然后从另一台（CPU）机器调用该服务，如下所示：

```
from bert_serving.client import BertClient

# 要带上参数 check_token_info=False，否则可能报错
bc = BertClient(ip='xx.xx.xx.xx', port=5555, port_out=5556, timeout=60000, check_version=False, check_token_info=False)
bc.encode(['First do it', 'then do it right', 'then do it better'])  #直接输入整个句子不需要提前分词
```

它将返回一个ndarray（或List[List[float]]如果您愿意的话），其中每一行都是代表一个句子的固定长度向量。

总结：Bert的输出最终有两个结果可用

sequence_output：维度【batch_size, seq_length, hidden_size】，这是训练后每个token的词向量。

pooled_output：维度是【batch_size, hidden_size】，每个sequence第一个位置CLS的向量输出，用于分类任务。



## 服务器和客户端API

### 服务器API

| 论据                    | 类型  | 默认               | 描述                                                         |
| ----------------------- | ----- | ------------------ | ------------------------------------------------------------ |
| `model_dir`             | str   | *Required*         | 预训练的BERT模型的文件夹路径。                               |
| `tuned_model_dir`       | str   | （可选的）         | 微调的BERT模型的文件夹路径。                                 |
| `ckpt_name`             | str   | `bert_model.ckpt`  | 检查点文件的文件名。                                         |
| `config_name`           | str   | `bert_config.json` | BERT模型的JSON配置文件的文件名。                             |
| `graph_tmp_dir`         | str   | None               | 图形临时文件的路径                                           |
| `max_seq_len`           | int   | `25`               | 最大序列长度，较长的序列将在右侧修剪。将其设置为NONE以动态使用（最小）批次中的最长序列。 |
| `cased_tokenization`    | bool  | False              | 标记生成器是否应跳过默认的小写字母和重音符号删除。应该用于例如多语言案例的预训练BERT模型。 |
| `mask_cls_sep`          | bool  | False              | 用零屏蔽[CLS]和[SEP]上的嵌入。                               |
| `num_worker`            | int   | `1`                | （GPU / CPU）工人数量运行BERT模型，每个工人在单独的进程中工作。 |
| `max_batch_size`        | int   | `256`              | 每个工人处理的最大序列数，较大的批次将被分成小批次。         |
| `priority_batch_size`   | int   | `16`               | 小于此大小的批处理将被标记为高优先级，并在作业队列中向前跳转以更快地获得结果 |
| `port`                  | int   | `5555`             | 用于将数据从客户端推送到服务器的端口                         |
| `port_out`              | int   | `5556`             | 用于将结果从服务器发布到客户端的端口                         |
| `http_port`             | int   | None               | 接收HTTP请求的服务器端口                                     |
| `cors`                  | str   | `*`                | 为HTTP请求设置“ Access-Control-Allow-Origin”                 |
| `pooling_strategy`      | str   | `REDUCE_MEAN`      | 用于产生编码矢量的合并策略，有效值为`NONE`，`REDUCE_MEAN`，`REDUCE_MAX`，`REDUCE_MEAN_MAX`，`CLS_TOKEN`，`FIRST_TOKEN`，`SEP_TOKEN`，`LAST_TOKEN`。这些策略的说明[可以在这里找到](https://github.com/hanxiao/bert-as-service#q-what-are-the-available-pooling-strategies)。要获得序列中每个令牌的编码，请将其设置为`NONE`。 |
| `pooling_layer`         | list  | `[-2]`             | 池在其上进行操作的编码层，其中`-1`表示最后一层，`-2`表示倒数第二层，`[-1, -2]`表示并置最后两层的结果，依此类推。 |
| `gpu_memory_fraction`   | float | `0.5`              | 每个工作人员应为每个GPU分配的内存总量的一部分                |
| `cpu`                   | bool  | False              | 在CPU而非GPU上运行                                           |
| `xla`                   | bool  | False              | 启用[XLA编译器](https://www.tensorflow.org/xla/jit)进行图形优化（*实验性！*） |
| `fp16`                  | bool  | False              | 使用float16精度（实验性）                                    |
| `device_map`            | list  | `[]`               | 指定将使用的GPU设备ID的列表（ID从0开始）                     |
| `show_tokens_to_client` | bool  | False              | 发送令牌化结果给客户端                                       |

注意：

bert_as_service默认的输出是 【倒数第二层】→【倒数第一层】 的隐藏状态的平均值！！！



### 客户端API

客户端提供了一个名为的Python类`BertClient`，该类接受如下参数：

| 论据                 | 类型 | 默认        | 描述                                                         |
| -------------------- | ---- | ----------- | ------------------------------------------------------------ |
| `ip`                 | str  | `localhost` | 服务器的IP地址                                               |
| `port`               | int  | `5555`      | 用于将数据从客户端推送到服务器的端口，*必须与服务器端配置一致* |
| `port_out`           | int  | `5556`      | 用于将结果从服务器发布到客户端的端口，*必须与服务器端配置一致* |
| `output_fmt`         | str  | `ndarray`   | 句子的输出格式编码为numpy数组或python List [List [float]]（`ndarray`/ `list`） |
| `show_server_config` | bool | `False`     | 首次连接时是否显示服务器配置                                 |
| `check_version`      | bool | `True`      | 是否强制客户端和服务器版本相同                               |
| `identity`           | str  | `None`      | 标识客户端的UUID，在多播中很有用                             |
| `timeout`            | int  | `-1`        | 设置客户端上接收操作的超时时间（毫秒）                       |

A `BertClient`实现以下方法和属性：

| 方法              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `.encode()`       | 将字符串列表编码为向量列表                                   |
| `.encode_async()` | 来自生成器的异步编码批处理                                   |
| `.fetch()`        | 从服务器获取所有编码的向量，并将其返回给生成器，将其与`.encode_async()`或结合使用`.encode(blocking=False)`。发送订单未保留。 |
| `.fetch_all()`    | 从服务器获取所有编码的向量，并将其返回到列表中，将其与`.encode_async()`或结合使用`.encode(blocking=False)`。发送订单被保留。 |
| `.close()`        | 优雅地关闭客户端和服务器之间的连接                           |
| `.status`         | 以JSON格式获取客户端状态                                     |
| `.server_status`  | 以JSON格式获取服务器状态                                     |



## 💡 想了解更多？

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
