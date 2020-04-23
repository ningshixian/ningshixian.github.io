---
layout:     post
title:      Python字符串模糊匹配
subtitle:   textdistance库的使用
date:       2020-03-25
author:     NSX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 技术
    - 教程
    - Python
---

# Python字符串模糊匹配

这篇文章将深入研究Python中的 [**textdistance**软件包](https://github.com/life4/textdistance)，该软件包提供了大量用于[模糊匹配](https://en.wikipedia.org/wiki/Approximate_string_matching)的算法。

## 介绍

最佳的字符串相似度算法是什么？好吧，要回答这个问题是很困难的，至少在不知道其他任何事情的情况下，例如您的要求。即使有了一个基本的想法，也很难在没有先在不同的数据集上对其进行尝试之前就找到一种好的算法。这是一个反复试验的过程。为了简化此过程，我尝试列出并解释最基本的字符串相似性算法的工作原理。试试看，这可能一直都是您需要的。

<!-- more -->

## 算法类型

根据操作的性质，字符串相似度算法可分为多个域。让我们讨论其中的一些，

- **基于编辑距离：**属于该类别的算法尝试计算将一个字符串转换为另一个字符串所需的操作数。操作数量越多，两个字符串之间的相似性就越少。需要注意的一点是，在这种情况下，字符串的每个索引字符都具有同等的重要性。

- **基于令牌的：**在此类别中，预期的输入是一组令牌，而不是完整的字符串。这个想法是在两个集合中找到相似的标记。公共令牌的数量越多，集合之间的相似性就越大。通过使用定界符进行分割，可以将字符串转换为集合。这样，我们可以将句子转换为单词或n-gram字符的记号。请注意，此处不同长度的令牌具有同等重要性。

- **基于序列：**这里，相似性是两个字符串之间公共子字符串的一个因素。该算法尝试找到两个字符串中存在的最长序列，找到的这些序列越多，相似性得分越高。注意，此处相同长度字符的组合具有同等重要性。



## textdistance库

**textdistance**包提供了可用于模糊匹配算法的集合，可以使用如下所示的pip来安装**textdistance**：

```
pip install textdistance
```

但是，如果您希望从算法中获得最快的速度，则可以调整pip install命令，如下所示：

```
pip install textdistance[extras]
```

安装完成后，我们可以像下面那样导入**textdistance**：

```
import textdistance
```

All algorithms have some common methods:

1. `.distance(*sequences)` -- calculate distance between sequences.
2. `.similarity(*sequences)` -- calculate similarity for sequences.
3. `.maximum(*sequences)` -- maximum possible value for distance and similarity. For any sequence: `distance + similarity == maximum`.
4. `.normalized_distance(*sequences)` -- normalized distance between sequences. The return value is a float between 0 and 1, where 0 means equal, and 1 totally different.
5. `.normalized_similarity(*sequences)` -- normalized similarity for sequences. The return value is a float between 0 and 1, where 0 means totally different, and 1 equal.

For example, [Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance):

```
import textdistance

textdistance.hamming('test', 'text')	# 1
textdistance.hamming.distance('test', 'text')	# 1
textdistance.hamming.similarity('test', 'text')	# 3
textdistance.hamming.normalized_distance('test', 'text')	# 0.25
textdistance.hamming.normalized_similarity('test', 'text')	# 0.75
textdistance.Hamming(qval=2).distance('test', 'text')	# 2
```

Any other algorithms have same interface.



## 基于编辑距离的算法

**汉明距离**

汉明距离是一个概念，它表示两个（相同长度）字对应位不同的数量，比如：1011101 与 1001001 之间的汉明距离是 2；2143896 与 2233796 之间的汉明距离是 3；"toned" 与 "roses" 之间的汉明距离是 3。

```
>> textdistance.hamming（'text'，'test'）
1 
>> textdistance.hamming.normalized_similarity（'text'，'test'）	# 将编辑距离限制在0和1之间
0.75 
```

**莱文斯坦距离**

莱文斯坦距离，又称**Levenshtein距离**，是[编辑距离](https://baike.baidu.com/item/编辑距离)的一种。指两个[字串](https://baike.baidu.com/item/字串)之间，由一个转成另一个所需的最少编辑操作次数。允许的编辑操作包括插入，删除和替换。莱文斯坦距离的应用：拼写检查、语音辨识、DNA分析、抄袭侦测。

```
textdistance.levenshtein("this test", "that test") # 2
textdistance.levenshtein.normalized_similarity("this test", "that test") # 0.8
```

**Jaro-Winkler距离**

Jaro-Winkler是两个字符串之间的另一个相似性度量。该算法对字符串中的字符串中的差异进行了惩罚。使用此算法的动机是，错字通常更有可能在字符串的后面而不是开头出现。比较"this test 和 test this"时，即使字符串包含完全相同的单词（只是顺序不同），相似度分数也仅为2/3。Jaro–Winkler distance 是适合于如名字这样较短的字符之间计算相似度。0分表示没有任何相似度，1分则代表完全匹配。*PS：在该算法中，更加突出了前缀相同的重要性，即如果两个字符串在前几个字符都相同的情况下，它们会获得更高的相似性。*

```
textdistance.jaro_winkler("this test", "test this") # .666666666...
textdistance.jaro_winkler(“ mes”，“ messi”) # 0.86 
```



## 基于令牌的算法

**Jaccard index**

Jaccard Similarity 度量两个字符串之间的共享字符，而不考虑顺序。公式是查找公用令牌的数量并将其除以唯一令牌的总数。用数学术语表示: A∩B / A∪B

```
# 使用默认的空格分隔符对字符串进行标记
>> tokens_1 =“ hello world” .split（）
>> tokens_2 =“ world hello” .split（）
>> textdistance.jaccard（tokens_1，tokens_2）
1.0 
>> tokens_1 =“ hello new world” .split（）
>> tokens_2 = “Hello World”的.split（）
>> textdistance.jaccard（tokens_1，tokens_2）
0.666
```

**Sørensen–Dice coefficient**

2|A∩B| / |A|+|B|

```
>> tokens_1 =“ hello world” .split（）
>> tokens_2 =“ world hello” .split（）
>> textdistance.sorensen（tokens_1，tokens_2）
1.0 
>> tokens_1 =“ hello new world” .split（）
>> tokens_2 =“ hello world” .split（）
>> textdistance.sorensen（tokens_1，tokens_2）
0.8
```

**余弦相似度**

余弦相似度是比较两个字符串的常用方法。该算法将字符串视为向量，并计算它们之间的余弦值。与上面的 Jaccard Similarity 相似，余弦相似度也忽略了要比较的字符串中的顺序。

```
textdistance.cosine("apple", "ppale") # 1.0
```

 

## 基于序列的算法

**Ratcliff-Obershelp相似性**

Find the longest common substring from the two strings. Remove that part from both strings, and split at the same location. This breaks the strings into two parts, one left and another to the right of the found common substring. Now take the left part of both strings and call the function again to find the longest common substring. Do this too for the right part. This process is repeated recursively until the size of any broken part is less than a default value. Finally, a formulation similar to the above-mentioned dice is followed to compute the similarity score.

```
>> string1, string2 = "i am going home", "gone home"
>> textdistance.ratcliff_obershelp(string1, string2)
0.66
>> string1, string2 = "helloworld", "worldhello"
>> textdistance.ratcliff_obershelp(string1, string2)
0.5
```



### **MRA（匹配评分方法）**

MRA（匹配评级方法）算法是一种语音匹配算法，即它尝试根据两个字符串的声音来测量它们之间的相似性。如果您要处理常见的拼写错误（在发音上没有太多损失），或者听起来相同但拼写不同的单词（谐音），则此算法可能很有用。它最初是用来比较听起来相似的名称的。例如，下面我们比较“ tie”和“ tye”。需要根据所涉及的字符串的长度将返回的分数与映射表进行比较（[有关更多详细信息，请参见此链接](https://en.wikipedia.org/wiki/Match_rating_approach)）。

```
textdistance.mra("tie", "tye") # 1
```



### 有关文本距离的更多信息

作为某些算法的默认设置，调用函数时，**textdistance**（与extras一起安装）将尝试查找外部库。其背后的目的是尝试以最佳速度实现实施。可以像下面这样关闭。

```
textdistance.jaro_winkler.external = False
 
textdistance.jaro_winkler("second test", "2nd test")
```

#### 模糊相似度百分比 

```
from fuzzywuzzy import fuzz
fuzz.ratio("this is a test", "this is a test!")
```



## 结论

字符串相似度算法的选择取决于用例。所有上述算法，以一种或另一种方式，都试图找到字符串的公共部分和非公共部分，并将它们分解为因子以生成相似度得分。而且，在不使过程复杂化的情况下，可以使用这些算法之一来解决大多数用例。更复杂的领域包括向量表示和压缩类型，它们也考虑了单词或n-gram的语义。



# 参考

[PYTHON模糊匹配指南](http://theautomatic.net/2019/11/13/guide-to-fuzzy-matching-with-python/)

[字符串相似性-基本了解算法指南！](https://itnext.io/string-similarity-the-basic-know-your-algorithms-guide-3de3d7346227)

