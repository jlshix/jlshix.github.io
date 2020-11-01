---
title: "使用 dir 和 getattr 实现分发模式"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - python
  - 设计模式
---

在 python 的[内置函数](https://docs.python.org/zh-cn/3/library/functions.html)中,
我们可以用 dir 和 getattr 实现分发.

- `dir([object])`: 返回某对象的有效属性名称列表

- `getattr(object, name[, default])`: `getattr(x, 'foobar')` 等同于 `x.foobar`


假设我们要实现一个检查类, 当调用此类的 `check()` 方法时, 将以无参的形式调用此类所有以
`check_` 开头的方法, 并将所有的值放置于一个列表中返回.

这是一个典型的 `分发, 搜集` 的需求, 在很多地方都会用到, python 内置的 unittest 也有类似
用法. 当调用 `unittest.main()` 时, 将会执行所有 `unittest.TestCase` 子类的以 `test_`
开头的方法. 当然 `unittest` 的内部实现不同, 有兴趣可自行学习.

如下是我的实现:

```python
class CheckMixin:
    """实现分发与搜集, 将以无参数的方式调用"""

    def check(self):
        """调用所有 `check_` 开头的方法, 搜集结果并返回"""
        rv = []
        for name in dir(self):
            if name.startswith('check_'):
                method = getattr(self, name)
                if callable(method):
                    resp = method()
                    rv.append(resp)
        return rv

# test below:

class Checker(CheckMixin):
    def check_01(self):     # pylint: disable=no-self-use
        return 'everything as normal'

    def check_02(self):     # pylint: disable=no-self-use
        return 'something wrong'

c = Checker()
assert c.check() == ['everything as normal', 'something wrong']

```

继承 `CheckMixin` 的子类都会获得一个 `check` 方法, 进行如下操作:

- 使用 `dir(self)` 得到所有的有效属性列表
- 遍历寻找 `check_` 开头的名称
- 获取名称对应的实例, 使用 [`callable`](https://docs.python.org/zh-cn/3/library/functions.html#callable) 检查其可调用性, 也可以使用 [inspect模块](https://docs.python.org/zh-cn/3/library/inspect.html#module-inspect)
- 满足条件则以无参的形式调用, 将结果加入返回列表中
- 返回结果

