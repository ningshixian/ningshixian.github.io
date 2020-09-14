---
title:      Simplify the Usage of Lexicon in Chinese NER
subtitle:   论文笔记
date:       2020-09-04
author:     NSX
catalog: true
tags:
    - 论文笔记

---

# NER论文：Simplify the Usage of Lexicon in Chinese NER

本篇论文为避免设计复杂的模型结构、同时为便于迁移到其他序列标注框架，提出了一种在embedding层简单利用词汇的方法。本文先后对比了三种不同的融合词汇的方式：

## **第一种：Softword**

![img](https://pic3.zhimg.com/80/v2-8924bacd6f5e4b98a1aea8ad51c01bfa_1440w.jpg)如上图所示，Softword通过中文分词模型后，对每一个字符进行BMESO的embedding嵌入。显而易见，这种Softword方式存在由分词造成的误差传播问题，同时也无法引入词汇对应word embedding。

## **第二种：ExtendSoftword**

![img](https://picb.zhimg.com/80/v2-55d78752c8d26c283c92d25fd1faa455_1440w.jpg)ExtendSoftword则将所有可能匹配到的词汇结果对字符进行编码表示。如上图中，对于字符「山」来说，其可匹配到的词汇有中山、中山西、山西路，「山」隶属 {B,M,E}。论文对于字符对应的词汇信息按照BMESO编码构建5维二元向量，如「山」表示为[1,1,1,0,0].

ExtendSoftword也会存在一些问题：1）仍然无法引入词汇对应的word embedding；2）也会造成信息损失，无法恢复词汇匹配结果，例如，假设有两个词汇列表[中山，山西，中山西路]和[中山，中山西，山西路]，按照ExtendSoftword方式，两个词汇列表对应字符匹配结果是一致的；换句话说，当前ExtendSoftword匹配结果无法复原原始的词汇信息是怎样的，从而导致信息损失。

## **第三种：Soft-lexicon**

![img](https://picb.zhimg.com/80/v2-ab558ef3c52fa9b4b2c5830a7a1d65b4_1440w.jpg)为了解决Softword和ExtendSoftword存在的问题，Soft-lexicon对当前字符，依次获取BMES对应所有词汇集合，然后再进行编码表示。

 <!-- more -->

![img](https://pic4.zhimg.com/80/v2-3226e5452411aa32381de3c140674412_1440w.jpg)

由上图可以看出，对于字符[语]，其标签B对应的词汇集合涵盖[语言，语言学]；标签M对应[中国语言]；标签E对应[国语、中国语]；标签S对应[语]。当前字符引入词汇信息后的特征表示为：


$$
\begin{array}{r}
e^{s}(\mathrm{B}, \mathrm{M}, \mathrm{E}, \mathrm{S})=\left[\boldsymbol{v}^{s}(\mathrm{B}) \oplus \mathrm{v}^{\mathrm{s}}(\mathrm{M}) \oplus \mathrm{v}^{\mathrm{s}}(\mathrm{E}) \oplus \mathrm{v}^{\mathrm{s}}(\mathrm{S})\right], \\
\mathbf{x}^{c} \leftarrow\left[\mathbf{x}^{c} ; \boldsymbol{e}^{s}(\mathrm{B}, \mathrm{M}, \mathrm{E}, \mathrm{S})\right]
\end{array}
$$


很容易理解，上述公式则将BMES对应的词汇编码 $V^s$ 与字符编码  $X^c$ 进行拼接。 $V^s$ 表示当前标签（BMES）下对应的词汇集合编码，其计算方式为：


$$
\boldsymbol{v}^{s}(S)=\frac{1}{Z} \sum_{w \in S} z(w) \boldsymbol{e}^{w}(w)
$$


S 为词汇集合， Z(w) 代表词频。考虑计算复杂度，本文没有采取动态加权方法，而采取如同上式的静态加权方法，即：对词汇集合中的词汇对应的word embedding通过其词频大小进行加权。词频根据训练集和测试集可离线统计。

综上可见，Soft-lexicon这种方法没有造成信息损失，同时又可以引入word embedding，此外，本方法的一个特点就是模型无关，可以适配于其他序列标注框架。



## 总结与对比：关键结果分析

基于如何更好的融入词汇信息为出发点，无外于两点：

1. 如何更充分的利用词汇信息、最大程度避免词汇信息损失；

2. 如何设计更为兼容词汇的Architecture，加快推断速度。



## 笔记

![](https://ningshixian.github.io/resources/images/Simplify-0001.jpg)



![](https://ningshixian.github.io/resources/images/Simplify-0002.jpg)



# 参考资料

https://blog.csdn.net/Wangpeiyi9979/article/details/102782954

[Simple-Lexicon：Simplify the Usage of Lexicon in Chinese NER（ACL2020）](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1908.05969.pdf)

[中文NER的正确打开方式: 词汇增强方法总结 (从Lattice LSTM到FLAT)](https://zhuanlan.zhihu.com/p/142615620)