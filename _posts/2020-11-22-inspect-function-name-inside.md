---
title: "使用 inspect 在函数中获取本身的名称"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - python
  - inspect
---

在类的某个实例中, 我们是可以获取本身的引用的, 也就是 `self`.

如果我们想获取 `self` 的类名可以这样: 

```python
class A:
    def cls_name(self):
        return self.__class__.__name__

a = A()
a.cls_name()

# 'A'
```

而在某个函数或者方法中, 我们就没办法了, 因为 python 显示持有自身的引用, 只提供的类的 `self`,
而没有函数或方法的 `self`.

所幸 python 有一个内置库 `inspect`, 提供了类型检查, 获取源代码, 检查类与函数, 检查解释器的
调用堆栈这四种主要的功能.


举例如下:

```python
import inspect


def somewhere():
    frame = inspect.currentframe()
    name = inspect.getframeinfo(frame).function
    return name


def who_am_i():
    stack = inspect.stack()
    return stack[1].function


def main():
    assert who_am_i() == 'main'
    assert somewhere() == 'somewhere'
    assert inspect.stack()[0].function \
           == inspect.getframeinfo(inspect.currentframe()).function \
           == 'main'


if __name__ == '__main__':
    main()

```

可以看到, 使用 `inspect` 可以拿到当前程序的调用栈, 栈的每一项为一个 `FrameInfo(frame, filename, lineno, function, code_context, index)`. 其中的 function 属性, 也就是
第三个, 可以拿到调用的函数的名字. 因此, 如果直接使用就是第 0 个, 如果通过调用别的函数使用就是
第 1 个.


参见:

- [the-interpreter-stack](https://docs.python.org/zh-cn/3/library/inspect.html#the-interpreter-stack)

- [inspect.currentframe](https://docs.python.org/zh-cn/3/library/inspect.html#inspect.currentframe)

- [inspect.getframeinfo](https://docs.python.org/zh-cn/3/library/inspect.html#inspect.getframeinfo)


- [Stack Overflow 上的回答](https://stackoverflow.com/questions/33162319/how-can-get-current-function-name-inside-that-function-in-python/33162432)