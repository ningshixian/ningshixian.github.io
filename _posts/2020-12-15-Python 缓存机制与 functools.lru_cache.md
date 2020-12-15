---
title:      Python3中的LRU缓存机制
subtitle:   functools.lru_cache
date:       2020-12-15
author:     NSX
catalog: true
tags:
    - Python
    - 缓存
    - Cache
---

缓存是一种将定量数据加以保存以备迎合后续获取需求的处理方式，旨在加快数据获取的速度。数据的生成过程可能需要经过计算，规整，远程获取等操作，如果是同一份数据需要多次使用，每次都重新生成会大大浪费时间。所以，如果将计算或者远程请求等操作获得的数据缓存下来，会加快后续的数据获取需求。

本文接下来会介绍Python3中的 `functools.lru_cache` 缓存机制以及对应的使用方法！

 <!-- more -->



# 前言

先来一个简单的例子以了解缓存机制的概念：

```
# -*- coding: utf-8 -*-

import random
import datetime


class MyCache:
    """缓存类"""

    def __init__(self):
        # 用字典结构以 kv 的形式缓存数据
        self.cache = {}
        # 限制缓存的大小，因为缓存的空间有限
        # 所以当缓存太大时，需要将旧的缓存舍弃掉
        self.max_cache_size = 10

    def __contains__(self, key):
        """根据该键是否存在于缓存当中返回 True 或者 False"""
        return key in self.cache

    def get(self, key):
        """从缓存中获取数据"""
        data = self.cache[key]
        data["date_accessed"] = datetime.datetime.now()
        return data["value"]

    def add(self, key, value):
        """更新该缓存字典，如果缓存太大则先删除最早条目"""
        if key not in self.cache and len(self.cache) >= self.max_cache_size:
            self.remove_oldest()
        self.cache[key] = {
            'date_accessed': datetime.datetime.now(),
            'value': value
        }

    def remove_oldest(self):
        """删除具备最早访问日期的输入数据"""
        oldest_entry = None

        for key in self.cache:
            if oldest_entry is None:
                oldest_entry = key
                continue
            curr_entry_date = self.cache[key]['date_accessed']
            oldest_entry_date = self.cache[oldest_entry]['date_accessed']
            if curr_entry_date < oldest_entry_date:
                oldest_entry = key

        self.cache.pop(oldest_entry)

    @property
    def size(self):
        """返回缓存容量大小"""
        return len(self.cache)


if __name__ == '__main__':
    # 测试缓存功能
    cache = MyCache()
    cache.add("test", sum(range(100000)))
    assert cache.get("test") == cache.get("test")

    keys = [
        'red', 'fox', 'fence', 'junk', 'other', 'alpha', 'bravo', 'cal',
        'devo', 'ele'
    ]
    s = 'abcdefghijklmnop'
    for i, key in enumerate(keys):
        if key in cache:
            continue
        else:
            value = ''.join([random.choice(s) for i in range(20)])
            cache.add(key, value)

    assert "test" not in cache
    print(cache.cache)
```

以上示例仅简单的展示了缓存机制的原理，通过用键值对的方式将数据放到字典中，如果下次需要取值时可以直接到字典中获取。该示例在删除旧数据时的实现并不高效，实际应用中可以用别的方式实现。

在 Python 的 3.2 版本中，引入了一个非常优雅的缓存机制，即 `functool` 模块中的 `lru_cache` 装饰器，可以直接将函数或类方法的结果缓存住，后续调用则直接返回缓存的结果。`lru_cache` 原型如下：

> @functools.lru_cache(maxsize=None, typed=False)

使用 functools 模块的 lur_cache 装饰器，可以缓存最多 maxsize 个此函数的调用结果，从而提高程序执行的效率，特别适合于耗时的函数。参数 `maxsize` 为最多缓存的次数，如果为 None，则无限制，设置为 2 的幂 时，性能最佳；如果 `typed=True`（注意，在 functools32 中没有此参数），则不同参数类型的调用将分别缓存，例如 f(3) 和 f(3.0)。

# LRU 算法介绍

**LRU (Least Recently Used，最近最少使用)** 算法是一种缓存淘汰策略。其根据数据的历史访问记录来进行淘汰，核心思想是，“如果数据最近被访问过，那么将来被访问的几率也更高”。该算法最初为操作系统中一种内存管理的页面置换算法，主要用于找出内存中较久时间没有使用的内存块，将其移出内存从而为新数据提供空间。其原理就如以上的简单示例。

那么问题来了，为什么LRU能提高性能？其实这个问题描述本身是错误的——LRU并不总 是能提高性能的，任何实用的缓存算法都不行。LRU基于这样一个前提：越久没被访 问的数据，以后被访问到的概率也越小。比方说，如果你的程序需要**周期性**地处 理不同数据，用LRU可能只会带来周期性的缓存miss从而增加处理器或者IO负担而已， 反而拖慢程序执行速度。

说到处理器负担，因为LRU要跟踪数据的访问时间/存活时间，通常涉及查找或者哈希 操作，所以需要更多处理器资源，有时候会很可观——

# functools里的LRU实现

以下为一个简单的 lru_cache 的使用效果：

```
from functools import lru_cache

@lru_cache(maxsize=32)
def add(x, y):
    print("calculating: %s + %s" % (x, y))
    return x + y

print(add(1, 2))
print(add(1, 2))
print(add(2, 3))
```

`@lru_cache(maxsize=32)` 中的 `maxsize` 参数就是缓存大小了，如果设成`None`，LRU逻辑会被禁用，变成一个 无限大的缓存；而如果设成`0`，缓存逻辑会被禁用，变成简单的调用次数统计。

输出结果：

```
calculating: 1 + 2
3
3
calculating: 2 + 3
5
```

从结果可以看出，当第二次调用 add(1, 2) 时，并没有真正执行函数体，而是直接返回缓存的结果。



# 还有什么需要注意的？

1. 目前的`lru_cache`是纯Python实现的。

2. 底层数据结构是普通的`list`和`dict`. `list`用于实现LRU链表，`dict`用于 查找已缓存的数据。在缓存已满的情况下，每次调用被缓存的函数时，都要进行 两次字典查找操作和20次以内的列表访问。

3. 对缓存的**所有**访问都是加了锁的，所以可以在多线程环境下使用。

4. 被 `lru_cache` 装饰的函数会有 `cache_clear` 和 `cache_info` 两个方法，分别用于清除缓存和查看缓存信息。

   ```
   >>> add.cache_info()          # 缓存信息
   CacheInfo(hits=0, misses=0, maxsize=10, currsize=0)
   >>> add.cache_clear()         # 清除所有缓存内容
   >>> add.__wrapped__           # 真正的 read_template 函数
   <function add at 0x7f9d0e9766a8>
   >>>
   ```

   返回访问数（hits）、未访问数（misses）和当前缓存使用量（currsize）、最大容量（maxsize）

5. **缓存的使用场景：**

   - 在缓存期内，数据不会更改。
   - 函数将始终为相同的参数返回相同的值（因此时间和随机对缓存没有意义）。
   - 函数没有副作用。如果缓存被访问，则永远不会调用该函数，因此请确保不更改其中的任何状态。
   - **函数不返回不同的可变对象。例如，返回列表的函数不适合缓存，因为将要缓存的是对列表的引用，而不是列表内容**（实际使用时，缓存的可变对象内容正确，不知道为啥？）



# 参考

[Python 缓存机制与 functools.lru_cache](http://kuanghy.github.io/2016/04/20/python-cache)

[Python3中的傻瓜式LRU缓存实现](https://blog.theerrorlog.com/simple-lru-cache-in-python-3-zh.html)

[如何让Python程序轻松加速，正确方法详解](https://my.oschina.net/u/4260786/blog/4263791)