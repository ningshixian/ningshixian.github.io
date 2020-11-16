---
title:      FAISS + SBERT向量语义检索解决方案
subtitle:   构建智能搜索引擎的原型
date:       2020-11-13
author:     NSX
catalog: true
tags:
    - Optuna
    - 超参数优化
    - 教程
---

![](https://ningshixian.github.io/resources/images/FAISS+SBERT.png)

# 介绍

语义搜索是一种信息检索系统，其重点是句子的含义，而不是常规的关键字匹配。基于关键词的搜索引擎通常会遇到以下问题:

- 复杂查询或具有双重含义的单词。

- 长查询，如论文摘要或博客中的一段。

- 不熟悉某个领域术语的用户或想要进行探索性搜索的用户。

基于向量(也称为语义)的搜索引擎通过使用最先进的语言模型找到文本查询的数字表示，在高维向量空间中对它们进行索引，并度量查询向量与索引文档的相似程度，从而解决了这些缺陷。

在本文中，我将讨论如何使用SOTA句子嵌入（[句子转换器](https://arxiv.org/pdf/1908.10084.pdf)）和[FAISS](https://github.com/facebookresearch/faiss)来实现最小语义搜索引擎，实现从海量文章中求topk相似文章。

FAQ 的处理流程一般为：

- **问题理解**，对用户 query 进行改写以及向量表示
- **召回模块**，在问题集上进行候选问题召回，获得 topk（基于关键字的倒排索引 vs 基于向量的语义召回）
- **排序模块**，对 topk 进行精排序

本项目着眼于 **召回模块** 的 **向量检索** 的实现，适用于 **小规模 FAQ 问题集**（候选问题集<10万）的系统快速搭建

 <!-- more -->



## FAQ检索方法

### 关键字检索（倒排索引）

传统召回模块**基于关键字检索**

- 计算关键字在问题集中的 [TF-IDF](https://en.wikipedia.org/wiki/Tf–idf) 以及[BM25](https://en.wikipedia.org/wiki/Okapi_BM25) 得分，并建立**倒排索引表**
- 第三方库 [ElasticSearch](https://whoosh.readthedocs.io/en/latest/index.html)，[Lucene](https://lucene.apache.org/pylucene/)，[Whoosh](https://whoosh.readthedocs.io/en/latest/index.html)

以Elasticsearch为例。Elasticsearch使用标记器将文档分割成标记(即有意义的文本单位)，这些标记映射到数字序列，并用于构建反向索引。

- 反向索引: 与检查每个文档是否包含查询词不同，反向索引使我们能够查找一个词并检索包含该词的所有文档列表。
- 同时，Elasticsearch用一个高维加权向量表示每个索引文档，其中每个不同的索引项是一个维度，它们的值(或权重)是用TF-IDF计算的。

- 在搜索过程中，使用相同的TF-IDF管道将查询转换为向量，文档d对查询q的VSM得分为加权查询向量V(q)和V(d)的余弦相似度。

### 向量检索（语义召回）

随着语义表示模型的增强、预训练模型的发展，基于 BERT 向量的**语义检索**得到广泛应用

- 对候选问题集合进行向量编码，得到 **corpus 向量矩阵**
- 当用户输入 query 时，同样进行编码得到 **query 向量表示**
- 然后进行语义检索（矩阵操作，KNN，FAISS）

针对小规模 FAQ 问题集直接计算 query 和 corpus 向量矩阵的**余弦相似度**，从而获得 topk 候选问题



**句向量获取解决方案**

| Python Lib                                                   | Framework  | Desc                                                    | Example                                                      |
| ------------------------------------------------------------ | ---------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| [bert-as-serivce](https://github.com/hanxiao/bert-as-service) | TensorFlow | 高并发服务调用，支持 fine-tune，较难拓展其他模型        | [getting-started](https://github.com/hanxiao/bert-as-service#getting-started) |
| [Sentence-Transformers](https://www.sbert.net/index.html)    | PyTorch    | 接口简单易用，支持各种模型调用，支持 fine-turn（单GPU） | [using-Sentence-Transformers-model](https://www.sbert.net/docs/quickstart.html#quickstart) [using-Transformers-model](https://github.com/UKPLab/sentence-transformers/issues/184#issuecomment-607069944) |
| 🤗 [Transformers](https://github.com/huggingface/transformers/) | PyTorch    | 自定义程度高，支持各种模型调用，支持 fine-turn（多GPU） | [sentence-embeddings-with-Transformers](https://www.sbert.net/docs/usage/computing_sentence_embeddings.html#sentence-embeddings-with-transformers) |



## SBERT +Faiss 语义搜索引擎

使用 `Sentence transformers` 对用户 query 进行向量表示，借助 `Faiss` 对问题集做相似性召回和打分，最后对 topk 结果进行精排序。

### Sentence transformers

Sentence transformers 是一个框架或者说是一系列的模型，用于将句子或段落表示成密集向量。由于BERT在这些任务中表现不佳，这些模型是 transformer 网络（BERT，RoBERTa等）专门针对语义文本相似性任务进行了微调。以下是STS基准测试中不同模型的性能：

![](https://ningshixian.github.io/resources/images/sbert排行榜.jpeg)

我们可以看到，Sentence transformer models 很大程度上优于其他模型。

但是，如果您通过[带有代码](https://paperswithcode.com/sota/semantic-textual-similarity-on-sts-benchmark)和[GLUE](https://gluebenchmark.com/leaderboard)的[论文](https://paperswithcode.com/sota/semantic-textual-similarity-on-sts-benchmark)查看排行榜，则会看到许多90以上的模型。那为什么我们需要Sentence transformers？

在这些模型中，语义文本相似性被视为回归任务。这意味着每当我们需要计算两个句子之间的相似性得分时，我们都需要将它们一起传递到模型中，然后模型输出它们之间的数字得分。尽管这对于基准测试非常有效，但对于实际使用案例却无法很好地扩展，这就是原因。

1. 当您需要搜索超过10k个文档时，您将需要执行10k个单独的推理计算，不可能分别计算嵌入并仅计算余弦相似度。参见作者的[解释](https://github.com/UKPLab/sentence-transformers/issues/405#issuecomment-689397806)。
2. 在两个文档之间共享最大序列长度（模型一次可以获取的单词/令牌的总数），这会导致由于分块而使表示形式被稀释

### Faiss

[Faiss](https://github.com/facebookresearch/faiss)是Facebook AI团队开源的针对聚类和相似性搜索的开源库，为稠密向量提供高效相似度搜索服务，支持十亿级别向量的搜索，是目前最为成熟的近似近邻搜索库之一。Faiss提供了多种索引类型如L2距离，向量內积等，详细介绍可参考[Faiss Indexes](https://github.com/facebookresearch/faiss/wiki/Faiss-indexes)，我们可以针对不同大小的向量集和聚类算法选择合适的索引类型。

> 它可对矢量化数据进行索引并对其进行高效搜索（建立向量索引，使用 k-nearest-neighbor 召回）

Faiss 可以根据以下 factors 提供不同的 indexes：

- search time
- search quality
- memory used per index vector
- training time
- need for external data for unsupervised training

因此，选择正确的 indexes 将是这些因素之间的权衡。

Faiss 总体使用过程可以分为三步：

1. 构建训练数据（以矩阵形式表达）
2. 挑选合适的 Index （Faiss 的核心部件），将训练数据 add 进 Index 中。
3. Search，也就是搜索，得到最后结果

### 详细步骤（附代码）

#### 1.加载SBERT模型并对文档进行矢量化

首先，让我们安装和加载所需的库

```
!pip install faiss-cpu
!pip install -U sentence-transformers

import numpy as np
import torch
import os
import pandas as pd
import faiss
import time
from sentence_transformers import SentenceTransformer
```

用**一百万个数据点**加载数据集

我使用了来自Kaggle的数据集，其中包含十七年来发布的新闻头条。

```
df=pd.read_csv("abcnews-date-text.csv")
data=df.headline_text.to_list()
```

加载预训练的模型并执行推理

```
model = SentenceTransformer('distilbert-base-nli-mean-tokens')
encoded_data = model.encode(data, show_progress_bar=True)
```

#### 2.Faiss索引数据集

通过参考[指南，](https://github.com/facebookresearch/faiss/wiki/Guidelines-to-choose-an-index)我们可以根据用例选择不同的索引选项。

让我们定义索引并向其中添加数据

```
index = faiss.IndexIDMap(faiss.IndexFlatIP(768))
index.add_with_ids(encoded_data, np.array(range(0, len(data))))

d, nlist = 768, 1000    # 聚类中心的个数

# # 精确的内积搜索，对归一化向量计算余弦相似度（不太准？）
# faiss.normalize_L2(encoded_data)   # 归一化
# index = faiss.IndexFlatIP(d)  # 内积建立索引
# index.add(encoded_data)  # 添加矩阵

# # 精确的L2距离搜索
# index = faiss.IndexFlatL2(d)
# print(index.is_trained)
# index.add(encoded_data) 
# print(index.ntotal)  # 查看建立索引的向量数目

# """
# 倒排文件检索
# 为了加速查询，可以把数据集切分成多个，采用基于Multi-probing(best-bin KD树变体）的分块方法。
# 这便是IndexIVFFlat，它需要另一个索引来记录倒排列表。
# """
# quantizer = faiss.IndexFlatIP(d)   # 建立一个量化器
# index = faiss.IndexIVFFlat(quantizer, d, nlist, faiss.METRIC_INNER_PRODUCT)
# index.train(encoded_data)
# index.add(encoded_data)
# print(index.ntotal)
```

序列化索引

```
faiss.write_index(index, 'abc_news')
```

然后可以将序列化的索引导出，迁移至托管搜索引擎的任何计算机中！

反序列化索引

```
index = faiss.read_index('abc_news')
```

#### 3.执行语义相似度搜索

首先让我们构建一个包装函数进行搜索

```
def search(query):
   t=time.time()
   query_vector = model.encode([query])
   k = 5
   top_k = index.search(query_vector, k)
   print('totaltime: {}'.format(time.time()-t))
   return [data[_id] for _id in top_k[1].tolist()[0]]
```

执行搜索

```
query=str(input())
results=search(query)
print('results :')
for result in results:
   print('\t',result)
```

#### 4.索引更新

https://juejin.im/post/6844903935044501511

每隔2分钟创建一个更新索引的task，要求更新所有handler对应的索引，worker接收到task之后从生产数据库上下载对应的主问题，下载对应的向量文件，加载向量构建索引，然后将索引序列化到文件，Faiss server提供gRPC接口来接收celery的通知(调用)，接到通知后直接加载索引文件更新索引。



## FAQ Web服务

### Web API

- Web 框架选择
  - [Flask](https://flask.palletsprojects.com/) + Gunicorn + gevent + nginx ，进程管理（崩溃自动重启）（uwsgi 同理，gunicorn 更简单）
  - :fire: **[FastAPI](https://fastapi.tiangolo.com/)** + uvicorn（崩溃自动重启），最快的Python Web框架（实测的确比 Flask 快几倍）
- cache 缓存机制（保存最近的query对应的topic，命中后直接返回）
  - Flask 相关
    - [flask-caching](https://github.com/sh4nks/flask-caching) （默认缓存500，超时300秒），使用 set/get 进行数据操作；项目来源于 [pallets/werkzeug](https://github.com/pallets/werkzeug) （werkzeug 版本0.4以后弃用 cache）
  - Python 3.2 以上自带（FastAPI 中可使用）
    - :fire: [**functools.lru_cache()**](https://docs.python.org/3/library/functools.html#functools.lru_cache) （默认缓存128，lru策略），装饰器，缓存函数输入和输出

### Locust 压力测试

使用 [Locust](https://locust.io/) 编写压力测试脚本

- 运行命令说明

  > 总共 100 个模拟用户，启动时每秒递增 10 个，压力测试持续 3 分钟

  ```
  locust  -f locust_test.py  --host=http://127.0.0.1:8889/module --headless -u 100 -r 10 -t 3m
  ```

- :hourglass: 配置 **4核8G CPU** （6层小模型占用内存约 700MB）

  - 小服务器上 **bert-as-service** 服务非常不稳定（tensorflow各种报错）， 效率不如简单封装的 **TransformersEncoder**
  - **FastAPI** 框架速度远胜于 **Flask**，的确堪称最快的 Python Web 框架
  - **cache** 的使用能够大大提高并发量和响应速度（最大缓存均设置为**500**）
  - 最终推荐配置 :fire: **TransformersEncoder + FastAPI + functools.lru_cache**



# 最后的想法

这是一个基本的实现，在语言模型部分和索引部分上仍然需要做很多工作。根据使用情况，数据大小和可用的计算能力，应从不同的索引选项中选择合适的索引选项。此外，本文使用的句子嵌入只是在某些公共数据集上进行微调，在特定领域的数据集上对其进行微调将改善嵌入效果，从而改善搜索结果。

很多场景下，基于关键字的倒排索引召回结果已经足够，可以考虑综合基于关键字和基于向量的召回方法，参考知乎语义检索系统 [Beyond Lexical: A Semantic Retrieval Framework for Textual SearchEngine](http://arxiv.org/abs/2008.03917)

# 参考文献

*[1] Nils Reimers和Iryna Gurevych。“*[*使用知识提炼使多语言的单语言句子嵌入成为多语言*](https://arxiv.org/pdf/2004.09813.pdf)*。” arXiv（2020）：2004.09813。*

[2] Johnson，Jeff和Douze，Matthijs和J {\'e} gou，Herv {\'e} *。“*[使用GPU进行十亿规模的相似性搜索](https://arxiv.org/abs/1702.08734)*”* arXiv预印本arXiv：1702.08734 *。*

[Fiass - 常见问题总结](https://zhuanlan.zhihu.com/p/107241260)

[Faiss 相似度搜索使用余弦相似性](https://blog.csdn.net/flyfish1986/article/details/108012151)

[FAQ之基于BERT的向量语义检索解决方案]([https://blog.khay.site/2020/09/06/FAQ%E4%B9%8B%E5%9F%BA%E4%BA%8EBERT%E7%9A%84%E5%90%91%E9%87%8F%E8%AF%AD%E4%B9%89%E6%A3%80%E7%B4%A2%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/](https://blog.khay.site/2020/09/06/FAQ之基于BERT的向量语义检索解决方案/))

[使用Sentence Transformers和Faiss构建语义搜索引擎](http://mp.163.com/article/FR52N3PU0531D9VR.html)