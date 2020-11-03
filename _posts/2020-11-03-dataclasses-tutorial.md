---
title: "初探 python 数据类"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - python
  - dataclass
---

## dataclass 是什么

数据类 `dataclass` 是很多编程语言中都有的概念, 一般用作与数据库中的数据结构映射, 封装
常用的方法, 实现更快捷的操作.

`dataclasses` 包是在 python 3.7 时加入标准库, 基于 [PEP 557](https://www.python.org/dev/peps/pep-0557/).

`dataclasses` 包提供了一个装饰器 `@dataclass` 和一系列函数, 用于在被装饰的类上添加
自动生成的特殊方法, 使其具有数据类的行为.

在 python 添加 `dataclasses` 之前, 最接近数据类这一用途的是 [`namedtuple`][1],
`dataclasses` 包的出现可以完全替代 `namedtuple` 并提供更为强大的功能.


## 使用 dataclass

若要使用, 首先导入 `dataclass` 装饰器:

```python
from dataclasses import dataclass
```

dataclass 的签名为:

```python
def dataclass(_cls=None, *, init=True, repr=True, eq=True, 
              order=False, unsafe_hash=False, frozen=False):
    pass
```

所以作为装饰器使用时以下三种方式等价:

```python
# 1. 直接使用
@dataclass
class A:


# 2. 无参数(即默认参数)调用
@dataclass()
class B:

# 3. 带参数调用
@dataclass(init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False)
class C:

```

举例:

```python
@dataclass
class Person(object):
    name: str
    age: int
    gender: str = '未填写'
```

`@dataclass` 装饰器会在被调用的 `Person` 类上添加 `__init__`, `__repr__`,
`__eq__` 方法. 如果 `Person` 类本身持有这些方法则不添加.

生成的 `__init__` 方法类似于:

```python
def __init__(self, name: str, age: int, gender: str = '未填写'):
    self.name = name
    self.age = age
    self.gender = gender
    self.__post_init__()
```

如果需要执行其他操作, 可以在 `__post_init__` 方法中进行.

所有生成的方法的参数的顺序为定义的顺序. 对于指定默认值有两点需要注意:

- 指定默认值的字段后不可有未指定默认值的字段, 否则会引发 `TypeError`

- 指定的默认值若为可变对象, 必须使用 `dataclasses.field` 方法的 default 参数指定

对于上例中定义的 `Person` 类, 如果不使用 `@dataclass` 修饰, `name age gender` 这三个
变量为类变量, 装饰之后为实例变量.

如果需要将某个变量定义为类变量, 则需要将其类型注解为 `typing.ClassVar`.

类似地, 如果某个变量被注解为 `dataclasses.InitVar`, 则此变量只作为 `__init__` 和
`__post_init__` 方法的参数, 不作为实例变量.

与 `dataclasses.InitVar` 相近地, 如果摸个变量在定义时设为 `dataclasses.field(init=False)`,
则此变量将不会出现在 `__init__` 方法的参数列表中, 需要在 `__post_init__` 方法中手动进行
初始化操作.


## dataclass 与 namedtuple

上文提到数据类可以替代命名元组, 因为数据类的功能更加强大, 在与命名元组的使用场景中,
数据类也提供了相似的功能:

1. 创建:
  - 使用 `Nt = namedtuple(typename, field_names)` 创建一个命名元组
  - 使用 `Dc = dataclasses.make_dataclass(cls_name, fields)` 创建一个数据类

2. 基于现有实例生成新实例:
  - 使用 `nt2 = nt1._replace(field1=new_value1)` 基于 nt1 替换 field1 的值生成 nt2
  - 使用 `dc2 = dataclasses.replace(dc1, field1=new_value1)` 基于 dc1 替换 field1 的值生成 dc2

3. 转为 tuple:
  - 使用 `tuple(nt1)` 得到 nt1 的值组成的元组
  - 使用 `dataclasses.astuple(dc1)` 得到 dc1 的值组成的元组, 还可指定 `tuple_factory` 参数.

4. 转为 dict:
  - 使用 `nt1._asdict()` 将 nt1 转为 OrderedDict 类型
  - 使用 `dataclasses.asdict(dc1)` 将 dc1 转为 dict 类型


## 总结

使用 `@dataclass` 及同 package 下的辅助函数可以在进行数据相关操作时减少大量样板代码,
更好地编写数据模型, 提高代码质量.

更多使用方法参见 [dataclasses 数据类][2]


[1]: https://docs.python.org/zh-cn/3/library/collections.html#collections.namedtuple
[2]: https://docs.python.org/zh-cn/3/library/dataclasses.html