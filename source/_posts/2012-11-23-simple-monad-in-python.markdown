---
layout: post
title: Python实现简单的Monad
date: 2012-11-23 22:06
comments: true
categories: [Python, Monad, Haskell]
---
今天看了一篇讲Ruby实现的Monad的文章（[这里][1]），所以一时兴起就也想用Python写一个。So，心动不如行动。。

在Haskell里，Monad就是用来把带有context的值应用到接受普通值并返回带context的值的函数的，用[Learn You a Haskell][2]话说就是

{% blockquote %}
If we have a fancy value and a function that takes a normal value but returns a fancy value, how do we feed that fancy value into the function? 
{% endblockquote %}

简单说，我们的目标就是Monad的两个主要的函数：

1.	return，把值放到最小化的context中
2.	\>>=（bind），把作为左操作数的Monadic值应用到作为右操作数的函数上，这个函数接受一个普通的值并返回Monadic值

首先，我们需要一个作为Monad的数据类型，这里简单用了一个继承自list的类MonadList，当然也可以自己造一个Maybe什么的。这样，return操作就是把值放到list里，不过Python已经把这个名字给占用了，所以只好取另一个名字wrap; 而bind就是把list里的元素逐一应用到给定的函数上（aka. map!），不过>>=在Python里是赋值语句，只能用个类似的>>了（不同于Haskell的Monad的>>）。

代码如下：
{% codeblock lang:python %}
#!/usr/bin/python

class MonadList(list):

    @staticmethod
    def wrap(value):
        return MonadList([value])

    def bind(self, f):
        return reduce(lambda acc, element: acc.extend(element) or acc,
                      map(f, self),
                      MonadList())

    def __rshift__(self, f):
        return self.bind(f)

    def __str__(self):
        return 'MonadList: ' + super(MonadList, self).__str__()


print MonadList([1, 2, 3, 4]) >> \
        (lambda x: MonadList.wrap(x ** 2)) \
        >> (lambda x: MonadList.wrap(x + 1))

{% endcodeblock %}

代码的作用很简单，就是把数组的元素先平方再加1 。。

另外，在写这篇文章的时候还发现了另一个[blog][3]，比较详细的讲了一个要产品化的代码里使用的Python的Monad，嗯哼。。

[1]: http://moonbase.rydia.net/mental/writings/programming/monads-in-ruby/00introduction “Monads in Ruby“
[2]: http://learnyouahaskell.com/ "Learn You a Haskell for Great Good!"
[3]: http://www.valuedlessons.com/2008/01/monads-in-python-with-nice-syntax.html "Monads in Python (with nice syntax!)"