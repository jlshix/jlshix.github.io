---
title: "Python glob 用法"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - Python
---

## glob: Unix 风格路径名模式扩展

glob 模块可根据 Unix 终端所用规则找出所有匹配特定模式的路径名, 但不能确定返回的顺序.

支持三种通配符:

- `*` 匹配 0 到多个字符
- `?` 匹配一个字符
- `[]` 匹配范围内的字符, 如 `[0-9]` 单个数字, `[A-Z]` 大写字母

常用的两个函数是 `glob` 和 `iglob`, 功能一致, 区别是返回匹配结果列表还是迭代器.

需要注意的是, 若目录中包含以 `.` 开头的文件, 默认不会被匹配.

默认不进行递归查找, 可以传入 `resursive=True` 参数开启.


## 示例

tmp 目录结构如下:

```text
tmp
├── ab
│   ├── 1.gif
│   ├── 2.gif
│   ├── 3.txt
│   └── card_2.txt
├── ac
│   └── card_1.txt
└── de
    └── summary.txt
```


可进行如下匹配:

```python
>>> from glob import glob, iglob
>>>
>>> glob("a*")
['ac', 'ab']
>>> glob("a*/*.txt")
['ac/card_1.txt', 'ab/3.txt', 'ab/card_2.txt']
>>>
>>>
>>> glob("ab/*.gif")
['ab/2.gif', 'ab/1.gif']
>>>
>>> glob("a*/card_*.txt")
['ac/card_1.txt', 'ab/card_2.txt']
>>>
>>> glob("ab/[0-9].*")
['ab/2.gif', 'ab/1.gif', 'ab/3.txt']
>>>
>>>
>>> glob("ab/?.gif")
['ab/2.gif', 'ab/1.gif']
```
## 参见

1. [glob --- Unix 风格路径名模式扩展](https://docs.python.org/zh-cn/3/library/glob.html#module-glob)