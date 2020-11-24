---
title: "pymongo 不存在则插入的实现"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - pymongo
  - mongodb
---

在执行插入操作时, 根据实际的场景有不同的需求, 通常有两种情况:

- 若存在则更新

- 若存在则放弃插入

我们以一条为例, 此时执行插入不再使用 `insert_one`, 而是使用 `update_one`.

我们可以看一下 [update_one](https://github.com/mongodb/mongo-python-driver/blob/807ab5ac9c153039b17bf5ffe4cd0d1d900c6631/pymongo/collection.py#L950) 方法的定义:

```python
def update_one(self, filter, update, upsert=False,
               bypass_document_validation=False,
               collation=None, array_filters=None, hint=None,
               session=None):
    pass
```

我们只需要了解 `filter`, `update` 和 `upsert` 三个参数即可.

- `filter`: 匹配文档的查询
- `update`: 执行的更改内容
- `upsert`: 若为 `True` 则在无匹配的情况下执行插入操作

当我们使用 `update_one` 时, 对于 `update` 参数, 我们通常指定的是 `{"$set": {"k1": "v1"}}`.

其实除了 `$set` 外, 还可以指定[其他的操作](https://docs.mongodb.com/manual/reference/operator/update-field/), 如 [`setOnInsert`](https://docs.mongodb.com/manual/reference/operator/update/setOnInsert/).

`$setOnInsert` 定义为: 如果执行更新操作时指定 `upsert=True`, 且确实执行了插入操作, 然后就
会执行 `$setOnInsert` 中指定的属性设置操作. 若未执行插入操作则什么也不做.

综上, 假设我们有一个 collection 名为 test, 其中的数据属性为 `key` 和 `value`.

回到一开始的两个问题, 给出对应的操作:

1. 若 `key` 存在则更新 `value`:

```python
db.test.update_one(
    filter={'key': '1'},
    update={'$set': {'value': 'some content'}}
)
```

2. 若 `key` 存在则放弃更新 `value`:

```python
db.test.update_one(
    filter={'key': '1'},
    update={'$setOnInsert': {'value': 'some content'}},
    upsert=True
)
```

即: 只有在找不到 `key` 为 '1' 的记录, 需要进行插入时才设定 `value` 的值为 'some content',
否则不进行任何操作, 符合 `key` 存在则放弃更新 `value` 的策略.

