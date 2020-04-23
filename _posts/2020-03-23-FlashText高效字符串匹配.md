---
layout:     post                    # 使用的布局（不需要改）
title:      FlashText高效字符串匹配               # 标题 
subtitle:   FlashText高效字符串匹配               #副标题
date:       2020-03-23              # 时间
author:     NSX                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 技术
    - 教程
    - Python
---


# FlashText高效字符串匹配

- 原文地址：[Regex was taking 5 days to run. So I built a tool that did it in 15 minutes.](https://link.juejin.im/?target=https%3A%2F%2Fmedium.freecodecamp.org%2Fregex-was-taking-5-days-flashtext-does-it-in-15-minutes-55f04411025f)



## 什么是 FlashText？

FlashText 是我在 [GitHub](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvi3k6i5) 上开源的一个 Python 库。它在提取关键词和替换上都很高效。

要使用 FlashText，首先必须传入一个关键词列表。此列表将在内部用于构建 Trie 字典。然后传入一个字符串，并说明是要替换还是搜索。

**替换**会创建一个替换了关键词的新字符串。**搜索**会返回字符串中找到的关键词列表。在运行过程中，输入的字符串只会被扫描一遍。

<!-- more -->

## 为什么 FlashText 这么快？

FlashText 算法基于 Aho-Corasick 算法和 Trie 数据结构。



## 什么时候应该使用 FlashText？

简单的答案：当关键词数 > 500 时

对于搜索，FlashText 在大约超过 500 个关键词后性能优于正则表达式。

复杂的答案：正则表达式可以基于特殊字符搜索关键词，如 `^,$,*,\d,.`，FlashText 不支持这个。

因此，如果您想像 `word\dvec` 这样匹配部分词，最好不要用 FlashText。但像 `word2vec` 这样提取完整的单词，就非常适合了。



## 导入词库

```
# 首先传入一个关键词列表。此列表将在内部用于构建 Trie 字典
kp = KeywordProcessor()
kp.add_keywords_from_list(project_list)
res_set = kp.extract_keywords(text)
```



## 用 FlashText 查找关键词

```
# pip install flashtext
from flashtext.keyword import KeywordProcessor
keyword_processor = KeywordProcessor()
keyword_processor.add_keyword('Big Apple', 'New York')
keyword_processor.add_keyword('Bay Area')
keywords_found = keyword_processor.extract_keywords('I love Big Apple and Bay Area.')
keywords_found
# ['New York', 'Bay Area']
```

FlashText 的简单提取示例



## 用 FlashText 替换关键词

您也可以替换句子中的关键词，而不是提取它。我们用这作为数据处理流程中的数据清理步骤。

```
from flashtext.keyword import KeywordProcessor
keyword_processor = KeywordProcessor()
keyword_processor.add_keyword('Big Apple', 'New York')
keyword_processor.add_keyword('New Delhi', 'NCR region')
new_sentence = keyword_processor.replace_keywords('I love Big Apple and new delhi.')
new_sentence
# 'I love New York and NCR region.'
```

使用 FlashText 的简单替换示例




# 参考

- [[译] 正则表达式要跑 5 天，所以我做了个工具，只跑 15 分钟。](https://juejin.im/post/5b6d426f6fb9a04fd1604341)

