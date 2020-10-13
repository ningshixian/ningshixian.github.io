---
title:      HuggingFace-Transformers手册
subtitle:   HuggingFace-Transformers手册
date:       2020-10-13
author:     NSX
catalog: true
tags:
    - 技术
    - 教程

---

### HuggingFace-Transformers手册

Transformers（以前称为pytorch Transformers和pytorch pretrained bert）为自然语言理解（NLU）和自然语言生成（NLG）提供了最先进的通用架构（bert、GPT-2、RoBERTa、XLM、DistilBert、XLNet、CTRL…），其中有超过32个100多种语言的预训练模型并同时支持TensorFlow 2.0和Pythorch两大深度学习框架。

 <!-- more -->



## 设计结构

预训练模型（TFPreTrainedModel）、模型配置（PretrainedConfig）、分词器（PreTrainedTokenizer）就是整个HuggingFace-Transformers的核心，它们分别负责模型结构和权重、模型的超参数、模型的输入预处理。
![img](https://yuanxiaosc.github.io/2019/12/30/HuggingFace-Transformers%E6%89%8B%E5%86%8C/HuggingFace-Transformers.svg)



## 使用教程

### 1、如何安装

transformers的安装十分简单，通过pip命令即可

```
pip install transformers
```



### 2、如何使用

使用transformers前需要下载好pytorch(版本>=1.0)或者tensorflow2.0。下面以pytorch为例，来演示使用方法

1、若要导入所有包可以输入：

```python
import torch
from transformers import *
```

2、若要导入指定的包可以输入：

```python
import torch
from transformers import BertModel
```

3、加载预训练权重和词表

```python
UNCASED = './bert-base-uncased'
bert = BertModel.from_pretrained(UNCASED)
```

注意：**加载预训练权重时需要下载好预训练的权重文件（pytorch版本），一般来说，当缓存文件中没有所需文件时(第一次使用)，只要网络没有问题，就会自动下载。当网络出现问题的时候，就需要手动下载预训练权重了。**

当缓存中不存在所需文件时，一般会出现提示：
bert-base-uncased-pytorch_model.bin not found in cache



## 参考

[https://yuanxiaosc.github.io/2019/12/30/HuggingFace-Transformers%E6%89%8B%E5%86%8C/](https://yuanxiaosc.github.io/2019/12/30/HuggingFace-Transformers手册/)

https://www.cnblogs.com/lian1995/p/11947522.html