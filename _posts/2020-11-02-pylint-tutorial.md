---
title: "pylint 入门"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - pylint
  - 代码检查
---

我们平常使用 IDE 编写 python 代码时, IDE 会使用 内置的代码静态检查工具来实时对我们的代码进行
检查, 及时发现问题并提示我们改正.

一般来讲, 代码检查工具都会实现 [PEP 8: Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/), 比如 `flake8` 等. 而 [pylint](https://www.pylint.org/) 在实现了 PEP8 的基础上更加激进地检查
你的代码并严厉地告诉你代码中的问题.

当然, 得益于 pylint 的高度可定制化, 我们可以在一次次的使用中了解我们并不需要的检查项,
将其全局禁用, 或在某个文件中禁用, 或在某处禁用.

若要在项目中使用 pylint, 首先需要安装:

```shell
pip install pylint
```

pylint 通过读取配置文件实现高度可定制化, 可将 pylint 的配置文件加入项目的代码版本管理中,
使其更适合当前的项目.

```shell
pylint --generate-rcfile > .pylintrc
```

得到一个配置文件, 在执行时使用 `--rcfile` 指定此配置.

pylint 输出的报告示例如下:

```shell
$ pylint --rcfile .pylintrc backend/main.py  
************* Module backend.main
backend/main.py:1:0: C0112: Empty module docstring (empty-docstring)
backend/main.py:15:0: C0116: Missing function or method docstring (missing-function-docstring)
backend/main.py:7:0: W0611: Unused MongoClient imported from pymongo (unused-import)
backend/main.py:10:0: W0611: Unused Doc imported from models.doc (unused-import)

-----------------------------------
Your code has been rated at 8.26/10
```

每条消息按顺序包含: `文件名:行号:第几个字符: 问题代码: 简短说明 (错误标识)`

若要全局禁用某个检查, 可在 `.pylintrc` 的 `disable` 项加入某错误标识.
若要在某文件中禁用, 则应在文件头部以注释行的形式注明: `# pylint: disable=错误标识`.
若要在某处禁用, 如 `main.py` 的第 1 行的最后添加 `pylint: disable=empty-docstring` 即可.

一个更常见的错误标识是 `invalid-name`, 而且通常是因为我们起的变量名太过简短, 如 `s`, `e`,
`rv`, `x`, `y` 等. 我们可以将这些名称加入 `good-names` 中, 使其不再报警.

如果你使用 `Pycharm` 作为 IDE, 还可以下载 [pylint插件](https://plugins.jetbrains.com/plugin/11084-pylint) 使用. 安装后重启就可以在项目中单独检查某文件, 某模块或整个项目, 将检查结果
展示在下方, 点击还可以跳转到对应位置, 方便修改.

