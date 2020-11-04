---
title: "使用 pytest 更好地进行测试"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - python
  - 测试
  - pytest
---

对代码进行测试有很多好处. 一份编写良好, 覆盖率高的测试就像一个按钮, 每次按下这个按钮都能知道代码的执行如你所想还是有意外发生.

但是编写并维护一份良好的测试是一个非常艰难的工作. 所以测试工具的选择能一定程度上影响编写测试的动力.

`pytest` 就是一个能提高编写并执行测试效率的工具.


## 为什么选择 pytest

如果你之前编写过单元测试, 那应该用过 python 内置的 `unittest` 模块.

内置的模块提供了一些基础的功能, 很多第三方框架致力于弥补这些缺点, 而 `pytest` 就是做得
最好的那个. `pytest` 的功能丰富, 写出来的代码更为简洁, 基于插件能使测试变得更方便.

更棒的是它开箱即用, 可以直接运行之前基于 unittest 模块编写的测试代码.

### 减少样板代码

大多数功能测试都遵循 `Arrage-Act-Assert` 模型, 即:
- 安排或设置测试条件
- 调用函数或方法执行测试
- 断言某些结束条件为真

断言失败时测试框架会介入并提供信息.

对于 unittest 来讲, 即便很小的测试也要编写相当数量的样板代码.
一般的流程是:

- 继承 `unittest.TestCase` 类
- 编写一系列的 `test_` 开头的方法
- 在方法中执行测试操作, 最后一般以 `self.assertEqual(got, expected)`
- 执行 `unittest.main()`

例如我们要测试一个函数

```python
# util.py
def odd_or_even(x):
    if x % 2:
        return 'odd'
    else:
        return 'even'
```

使用 unittest 编写测试:

```python
# test_util.py

from unittest import TestCase, main

from utils import odd_or_even

class TestOddEven(TestCase):
    def test_odd(self):
        self.assertEqual(odd_or_even(3), 'odd')
    
    def test_even(self):
        self.assertEqual(odd_or_even(2), 'even')
    
    def test_raise(self):
        with self.assertRaises(TypeError):
            odd_or_even('1')
    
if __name__ == '__main__':
    main()
```

然后在命令行执行 `python -m unittest discover`


而 `pytest` 只需要:

```python
# test_util.py

import pytest

from util import odd_or_even

def test_odd():
    assert odd_or_even(3) == 'odd'

def test_even():
    assert odd_or_even(2) == 'even'

def test_raise():
    with pytest.raises(TypeError):
        odd_or_even('1')
```

然后在命令行直接执行 `pytest` 即可.


### 使用 `@fixture` 显式展现依赖

在 `unittest` 中使用 `setUp` 方法提前准备各个 `test_` 方法所需要的依赖.
但具体到某个函数时其依赖的展现是隐式的.

而 `pytest` 使用 `@fixture` 修饰一个函数, 在运行测试之前执行此函数保存其返回值.
当测试方法中定义的参数与此函数同名时, 将其值作为参数传入. 实现显式地展现每个测试函数的依赖.

```python
# conftest.py

import pytest

from pymongo import MongoClient

@pytest.fixture
def db():
    client = MongoClient()
    yield client.get_database('test')
    # yield 及之前为 setUp, 之后为 tearDown
    rv.close()


# tests.py

def test_count(db):
    col = db.get_collection('test')
    assert col.count_documents({}) == 2
```

### 标记

可以为每个测试函数使用 `@pytest.mark` 创建标记, 在运行时只允许带有某标记的用例,
从而进行快速的测试.


## 总结

`pytest` 是一款优秀的第三方测试框架, 弥补了很多 `unittest` 模块的不足. 使用 `pytest`
可以让你更有动力去为项目编写一套良好的测试用例集.

快使用 `pip install pytest` 安装使用吧 :)


## 参考

- [pytest documentation](https://docs.pytest.org/en/stable/)

- [pytest-python-testing](https://realpython.com/pytest-python-testing/)