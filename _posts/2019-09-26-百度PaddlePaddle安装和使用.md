---
layout:     post                    # 使用的布局（不需要改）
title:      百度PaddlePaddle安装和使用               # 标题 
subtitle:   百度PaddlePaddle安装和使用               #副标题
date:       2019-09-26              # 时间
author:     NSX                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 技术
    - 教程
---


# 百度PaddlePaddle快速安装

[win环境准备](https://www.paddlepaddle.org.cn/install/doc/source/windows)

- Windows 7/8/10 专业版/企业版 (64bit) (CPU版本)
- Python 版本 2.7/3.5.1+/3.6/3.7 (64 bit)
- pip 或 pip3 版本 9.0.1+ (64 bit)
- Visual Studio 2015 Update3

[linux环境准备](https://www.paddlepaddle.org.cn/install/doc/source/centos)

- CentOS 版本 (64 bit)
- CentOS 6 (不推荐，不提供编译出现问题时的官方支持)
- CentOS 7 (GPU 版本支持CUDA 9/10.0)
- Python 版本 2.7.15+/3.5.1+/3.6/3.7 (64 bit)
- pip 或 pip3 版本 9.0.1+ (64 bit)

<!-- more -->

### 1. 下载源码+编译安装

清华源提供的 `paddlepaddle` 列表，在[这里](https://pypi.tuna.tsinghua.edu.cn/simple/paddlepaddle/)查看

- python3.6  linux
  - paddlepaddle-1.6.0rc0-cp36-cp36m-manylinux1_x86_64.whl  [下载地址](https://pypi.tuna.tsinghua.edu.cn/packages/3c/25/7d8b66bef57e86e08c73844958fea5d6b7e931b30abc488ec7a37fd8bbce/paddlepaddle-1.6.0rc0-cp36-cp36m-manylinux1_x86_64.whl#sha256=9a4bf72b6e70ac94f3cbd828bd514c1d7ad58052a792ccf9e0a5371e18bf9e88)
  - paddlepaddle-1.6.1-cp36-cp36m-manylinux1_x86_64.whl   [下载地址](https://pypi.tuna.tsinghua.edu.cn/packages/f1/48/3531a6b6ea5e23ae3e0a362e63f59679022baa82f06078e11232683df830/paddlepaddle-1.6.1-cp36-cp36m-manylinux1_x86_64.whl#sha256=bcc3949212f17035f5bcd7cac2dbbd0dbc158d9446344fe2237d76e24958cef1)

- python3.7  linux
  - paddlepaddle-1.6.0rc0-cp37-cp37m-manylinux1_x86_64.whl  [下载地址](https://pypi.tuna.tsinghua.edu.cn/packages/1e/e7/00eb8402471f5057c525e86bc4cde669f3c0400b1a37c8344e6d76cca052/paddlepaddle-1.6.0rc0-cp37-cp37m-manylinux1_x86_64.whl#sha256=590af137d8975e3d54583b62e2383e61320a304ece2dcdf8e4c40858bcfa161b)
  - paddlepaddle-1.6.1-cp37-cp37m-manylinux1_x86_64.whl  [下载地址](https://pypi.tuna.tsinghua.edu.cn/packages/c5/05/106ac422a366df64645cc9705e02cf5c297a0937becbe073ccad3900a8f6/paddlepaddle-1.6.1-cp37-cp37m-manylinux1_x86_64.whl#sha256=618c4bcfddb56055a2abe56c814074e9c13bb893efd416d86a3dd1e312287ae8)



### 2. 在当前机器或目标机器安装编译好的 .whl 包：

```
pip install -U（whl包的名字）
pip3 install -U（whl包的名字）
```



### 3. 修改 paddlepaddle 源码，获取错误信息

1. 备份module文件：
```
cp /home/ningshixian/.paddlehub/modules/lac/python/a62c5d015111daae0dbd8719b0293c1b.py /home/ningshixian/.paddlehub/modules/lac/python/a62c5d015111daae0dbd8719b0293c1b.py.bak
```
2. 获取异常信息
```
vi /home/ningshixian/.paddlehub/modules/lac/python/a62c5d015111daae0dbd8719b0293c1b.py
```
​	将304行cur_word = words[word_index]修改为：

```
try:
    cur_word = words[word_index]
except Exception as exc:
    raise Exception("Exception: %s, words: %s, word_index: %s, lod_info:%s, crf_decode:%s"%(exc,words,word_index,lod_info,crf_decode))
```

恭喜，至此您已完成PaddlePaddle的编译安装!!



### 4. 验证安装

安装完成后您可以使用 python 或 python3 进入python解释器，输入 `import paddle.fluid as fluid`，再输入 `fluid.install_check.run_check()`，如果出现 `Your Paddle Fluid is installed succesfully!`，说明您已成功安装。



### 5. [paddleHub 快速安装](https://github.com/PaddlePaddle/PaddleHub)

PaddleHub是基于PaddlePaddle生态下的预训练模型管理和迁移学习工具，可以结合预训练模型更便捷地开展迁移学习工作。

> 环境依赖
>
> - Python==2.7 or Python>=3.5
> - PaddlePaddle>=1.4.0

pip安装方式如下：```$ pip install paddlehub```



### 6. [词法分析LAC模型下载安装](https://www.paddlepaddle.org.cn/hubdetail?name=lac&en_category=LexicalAnalysis)

> Lexical Analysis of Chinese，简称 LAC，是一个联合的词法分析模型，能整体性地完成中文分词、词性标注、专名识别任务。

PaddleHub中的预训练模型和预置数据集都需要通过服务端进行下载，因此PaddleHub默认用户访问外网权限。

```hub install lac==2.0.0```

安装位置：` C:\Users\ningshixian\.paddlehub\cache\lac`

[https://github.com/PaddlePaddle/models/tree/develop/PaddleNLP/lexical_analysis](https://github.com/PaddlePaddle/models/tree/develop/PaddleNLP/lexical_analysis)



### 7. paddlehub使用代码示例

```python
import paddlehub as hub

lac = hub.Module(name="lac")
test_text = ["今天是个好日子", "天气预报说今天要下雨", "下一班地铁马上就要到了"]

inputs = {"text": test_text}
results = lac.lexical_analysis(data=inputs)

for result in results:
    print(result['word'])
    print(result['tag'])
```

参数

> data：dict类型，key为text，str类型，value为待分词文本，list类型
> user_dict: 用户自定义词典的路径，当提供自定义词典时，用户可以通过自定义词典来干预分词结果。
> 词典包含三列，第一列为单词，第二列为单词词性，第三列为单词词频，以水平制表符分隔。词频越高的单词，对分词结果影响越大

返回

> results：list类型，每个元素为对应输入文本的预测结果。
> 预测结果为dict类型，有word和tag字段， word字段存放文本分词后的各个单词，tag存放各个单词对应的标签(`tag=='nr' or tag=='PER'`)



# 参考

- [paddlehub官方github](https://github.com/PaddlePaddle/PaddleHub)

- [lac](https://www.paddlepaddle.org.cn/hubdetail?name=lac&en_category=LexicalAnalysis)
