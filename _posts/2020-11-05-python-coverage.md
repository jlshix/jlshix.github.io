---
title: "使用 coverage 分析测试代码覆盖率"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - python
  - 测试
  - coverage
---

在测试中, 测试用例执行时覆盖了功能代码行数的百分比被称为覆盖率. 是衡量测试有效性及产品
质量的标准之一.

而 coverage 是 python 代码计算覆盖率的最佳工具. 使用 coverage 从指定的测试入口执行测试,
coverage 会实时监控代码的执行情况, 执行完毕后会告知哪些行执行到了, 哪些行没有被执行到.


## 安装 coverage

执行 `pip install coverage` 即可.

## 执行

使用 `coverage run` 命令执行测试套件并搜集数据. 如果你使用 `pytest` 作为测试套件,
可以执行: `coverage run -m pytest arg1 arg2 ...`, 后面的 `arg1`, `arg2` 等
为 pytest 命令的参数.

当然, 为了限制 `coverage` 只搜集当前 package 的代码覆盖率情况, 可以使用 `--source=.`
限制统计范围为当前文件夹.


## 获取报告

`coverage run` 执行完成后会在当前文件夹下生成一个 `.coverage` 文件, 但不能直接打开, 可以
使用 `coverage report` 命令查看结果, 类似于如下输出:

```shell
$ coverage report -m
Name                      Stmts   Miss  Cover   Missing
-------------------------------------------------------
my_program.py                20      4    80%   33-35, 39
my_other_module.py           56      6    89%   17-23
-------------------------------------------------------
TOTAL                        76     10    87%
```

如果想要更好的可读性也可执行 `coverage html` 查看生成网页版本的报告查看.


## 与 pycharm 集成

pycharm 的 coverage 是随时可用的. 当我们需要运行某个文件时, 运行选项中总是会有一项
`run 'filename' with Coverage`. 运行完毕后会默认在右侧弹出一个覆盖率窗口列出每个
源文件的覆盖率. 同时在编辑器窗口也会以类似版本控制风格的形式标出哪些行覆盖到了, 哪些没有.

此外, 也可以与 CI 系统整合, 之后会更新相关文章.

## 参见

- [coverage.py 5.3 documentation](https://coverage.readthedocs.io/en/coverage-5.3/index.html)

最后放上 coverage 的吉祥物: Sleepy Snake

![Sleepy Snake](https://coverage.readthedocs.io/en/coverage-5.3/_images/sleepy-snake-600.png)