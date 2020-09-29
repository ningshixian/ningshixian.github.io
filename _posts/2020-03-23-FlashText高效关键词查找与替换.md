---
layout:     post                    # 使用的布局（不需要改）
title:      FlashText高效关键词查找与替换               # 标题 
subtitle:   FlashText算法可用于大规模替换、检索文档中的关键字               #副标题
date:       2020-03-23              # 时间
author:     NSX                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 技术
    - 教程
    - Python
---

# FlashText高效关键词查找与替换

> FlashText算法可用于大规模替换、检索文档中的关键字
>
> 可用于槽位提取中的关键词实体抽取！
>
> Github：https://github.com/vi3k6i5/flashtext



## 什么是 FlashText？

正则表达式在一个 10k 的词库中查找 15k 个关键词的时间差不多是 0.165 秒。但是对于 Flashtext 而言只需要 0.002 秒。因此，在这个问题上 Flashtext 的速度大约比正则表达式快 82 倍。

随着我们需要处理的字符越来越多，正则表达式的处理速度几乎都是线性增加的。然而，Flashtext 几乎是一个常量。

<!-- more -->



## 为什么 FlashText 这么快？

Flashtext 是一种基于 Trie 字典数据结构和 Aho Corasick 的算法。

它的工作方式是，首先它将所有相关的关键字作为输入。使用这些关键字建立一个 trie 字典，这个 trie 字典就是我们后面要用来搜索和替换的数据结构。



## 什么时候应该使用 FlashText？

简单的答案：当关键词数 > 500 时

对于搜索，FlashText 在大约超过 500 个关键词后性能优于正则表达式。

复杂的答案：正则表达式可以基于特殊字符搜索关键词，如 `^,$,*,\d,.`，FlashText 不支持这个。

因此，如果您想像 `word\dvec` 这样匹配部分词，最好不要用 FlashText。但像 `word2vec` 这样提取完整的单词，就非常适合了。



## 使用方法

### 安装

```
pip install flashtext
```

 

### 导入词库

首先传入一个关键词列表。此列表将在内部用于构建 Trie 字典

```python
from flashtext.keyword import KeywordProcessor

kp = KeywordProcessor()
keyword_dict={
            "fruit": ["apple", "banana","orange","watermelon"], 
            "ball": ["tennis", "basketball","football"]
             }
keyword_processor.add_keywords_from_dict(keyword_dict)         # 可以添加dict
keyword_processor.add_keywords_from_list(["fruit", "banana"])  # 可以添加list
```



### 关键词提取

```python
from flashtext.keyword import KeywordProcessor

keyword_processor = KeywordProcessor(case_sensitive=False)
keyword_processor.add_keyword('Big Apple', 'New York')
keyword_processor.add_keyword('Bay Area')
keywords_found = keyword_processor.extract_keywords('I love Big Apple and Bay Area.',span_info=True)
keywords_found
# ['New York', 'Bay Area']
```

其中：

- case_sensitive，是否对大小写敏感
- span_info，是否要返回位置信息



### 关键词替换

您也可以替换句子中的关键词，而不是提取它。我们用这作为数据处理流程中的数据清理步骤。

```Python
from flashtext.keyword import KeywordProcessor

keyword_processor = KeywordProcessor()
keyword_processor.add_keyword('Big Apple', 'New York')
keyword_processor.add_keyword('New Delhi', 'NCR region')
new_sentence = keyword_processor.replace_keywords('I love Big Apple and new delhi.')
new_sentence
# 'I love New York and NCR region.'
```



### 删除关键词

```python
keyword_processor.remove_keyword('banana')
keyword_processor.remove_keywords_from_dict({"food": ["bread"]})
keyword_processor.remove_keywords_from_list(["basketball"])
123456
```




# 参考

- [[译] 正则表达式要跑 5 天，所以我做了个工具，只跑 15 分钟。](https://juejin.im/post/5b6d426f6fb9a04fd1604341)

- [Regex was taking 5 days to run. So I built a tool that did it in 15 minutes.](https://link.juejin.im/?target=https%3A%2F%2Fmedium.freecodecamp.org%2Fregex-was-taking-5-days-flashtext-does-it-in-15-minutes-55f04411025f)

- [FlashText – A library faster than Regular Expressions for NLP tasks](https://www.analyticsvidhya.com/blog/2017/11/flashtext-a-library-faster-than-regular-expressions/)

- [Flashtext：大规模数据清洗的利器](https://blog.csdn.net/CoderPai/article/details/78574863)

- [超大规模文本数据清洗、查找、匹配神器之python模块flashtext学习使用](https://blog.csdn.net/Together_CZ/article/details/82693099)