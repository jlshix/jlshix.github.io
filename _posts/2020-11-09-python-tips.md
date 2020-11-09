---
title: "python tips"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - python
  - tips
---

记录一些小技巧.

## 获取子类

有时需要寻找某个类的所有直接子类用于分发, 以前的惯用法是在 package 的 `__init__.py`
内提前导入. 其实可以找到那个类, 直接调用其 `__subclasses__()` 方法就可以得到一个
子类的列表.

参见:

- https://stackoverflow.com/questions/3862310/how-to-find-all-the-subclasses-of-a-class-given-its-name






