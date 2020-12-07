---
title:      几个简单的Python问题tips
subtitle:   Python中可变对象和不可变对象之间的区别
date:       2020-06-28
author:     NSX
catalog: true
tags:
    - Python
    - zip(*list)
---

尝试解决以下问题，然后检查以下答案：

- 问题1：
- 问题1
- 问题1

<!-- more -->



# 问题1

假设我们有两个变量：

```
x = 1
y = 2
l = [x, y]
x += 5

a = [1]
b = [2]
s = [a, b]
a.append(5)
```

l和s的输出是什么？

[跳到解决方案1](#解决方案1)



# 问题2

让我们定义一个简单的函数：

```
def f(x, s=set()):
    s.add(x)
    print(s)
```

What will happen if you call:

```
>>f(7)
>>f(6, {4, 5})
>>f(2)
```

[跳到解决方案2](#解决方案2)



# 问题3

让我们定义两个简单的函数：

```
def f():
    l = [1]
    def inner(x):
        l.append(x)
        return l
    return inner
    
def g():
    y = 1
    def inner(x):
        y += x
        return y
    return inner
    
def h():
    l = [1]
    def inner(x):
        l += [x]
        return l
    return inner
```

以下命令将产生什么结果？

```
>>f_inner = f()
>>print(f_inner(2))

>>g_inner = g()
>>print(g_inner(2))

>>h_inner = h()
>>print(h_inner(2))
```

[跳到解决方案3](#解决方案3)



# 问题4

[Difference between zip(list) and zip(*list)](https://stackoverflow.com/questions/29139350/difference-between-ziplist-and-ziplist/29139418) 

```
p = [[1,2,3],[4,5,6]]

>>>d=zip(p)
>>>list(d)
[([1, 2, 3],), ([4, 5, 6],)]

>>>d=zip(*p)
>>>list(d)
[(1, 4), (2, 5), (3, 6)]
```

Answer:

`zip` wants a bunch of arguments to zip together, but what you have is a single argument (a list, whose elements are also lists). The `*` in a function call "unpacks" a list (or other iterable), making each of its elements a separate argument. So without the `*`, you're doing `zip( [[1,2,3],[4,5,6]] )`. With the `*`, you're doing `zip([1,2,3], [4,5,6])`.



-----



# 解决方案1

```
>>print(l)
[1, 2]

>>print(s)
[[1, 5], [2]]
```

由于`x`是不可变的，因此该操作`x+=5`不会更改原始对象，而是会创建一个新对象。列表的第一个元素仍指向原始对象，因此其值保持不变。

如果对象是可变的`a`，则`a.append(5)`更改原始对象，因此列表`s` 的输出变了！



# 解决方案2

```
>>f(7)
{7}

>>f(6, {4, 5})
{4, 5, 6}

>>f(2)
{2, 7}
```

前两个输出是完全有意义的：首先将的值`7`添加到默认的空集结果中`{7}`，然后将的值`6`添加到`{4, 5}`结果集中`{4, 5, 6}`。

但是随后发生了一件奇怪的事情：的值`2`不是添加到默认的空集，而是添加到的集`{7}`。为什么？**可选参数s的默认值仅被评估一次**-仅在第一次调用期间`s`将被初始化为空集。因为在调用后`s`是**可变的**，`f(7)`所以对其进行了修改。第二个调用`f(6, {4, 5})`不会影响默认参数-提供的设置将其`{4, 5}`隐藏，换句话说，它`{4, 5}`是一个不同的变量。第三次调用`f(2)`使用的`s`变量与第一次调用中使用的变量相同，但不会将s重新初始化为空集，而是使用s的先前值`{7}`。

这就是为什么 **你不应该使用可变的默认参数**。在这种情况下，应按以下方式修改功能：

```
def f(x, s=None):
    if s is None:
        s = set()
    s.add(x)
    print(s)
```



# 解决方案3

```
>>f_inner = f()
>>print(f_inner(2))
[1, 2]

>>g_inner = g()
>>print(g_inner(2))
UnboundLocalError: local variable ‘y’ referenced before assignment

>>h_inner = h()
>>print(h_inner(2))
UnboundLocalError: local variable ‘l’ referenced before assignment
```

In this problem we deal with **closures** — the fact that inner functions remember how their enclosing namespaces looked at the time of definition.

So what is happening? If you compared just the behavior of the first two functions, you could think that once again the difference has to do with `l` being a mutable list and `x` being an immutable string.

Looking at the example with the function `h` shows that there is something else to it. Here just like in the function `f` we are trying to alter a mutable list, but this time we fail. You would think that operations `l.extend([x])` and`l += [x]` are equivalent, but in fact there is a subtle difference between them: behind the scenes `l.extend` will cause compiler to call `LOAD_DEREF` instruction which will load the non-local list `l` (and then the list will be modified), while with `l += [x]` the compiler will call `LOAD_FAST` which looks for the variable `l` to in the local scope of the inner function and therefore fails.



# **结论**

**注意可变变量和不可变变量之间的区别。**

**不要使用可变的默认参数。**

**在内部函数中更改闭包变量时要小心。使用nonlocal关键字声明该变量不是本地变量。**



# 参考

[Can you solve these 3 (seemingly) easy Python problems?](https://levelup.gitconnected.com/can-you-solve-these-3-seemingly-easy-python-problems-2c793967cd2c#42ad)