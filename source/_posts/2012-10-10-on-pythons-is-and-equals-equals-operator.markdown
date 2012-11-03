---
layout: post
title: "Python的is和==操作符用法"
date: 2012-10-10 23:44
comments: true
categories: [Python]
---
我只是想说下今天代码检查中遇到的一个问题，不做详细的说明。代码里有这种用法：

```python
if someVar is []:
    blah
…
if anotherVar is 1:
    blah
```

很明显这么用是错的，判断空链表应该用隐式转换，值是否为1用==。当然为了说明清楚问题，稍微研究了一下。控制台里进行了这样的测试：

```python
>>> 1 == 1
True
>>> n = 1
>>> n is 1
True
>>> a = 1000
>>> a is 1000
False
>>> [] is []
False
>>> [] == []
True
>>> 
```

`n is 1`返回True是由于Python对于小整数的优化（小整数都是预先创建好的，需要时直接使用），`a is 1000`返回False是因为一般的整数都是需要时直接创建的对象并且就算之前有相同的值的对象也不会复用。不过如果写`1000 is 1000`是会返回True的，因为这俩字面常量实际上引用了一个对象，这应该是编译器的优化吧。`[] is []`返回False就显而易见了，因为每个list都是全新的对象，不会复用已有的对象。