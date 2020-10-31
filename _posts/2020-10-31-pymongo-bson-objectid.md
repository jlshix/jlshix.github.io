---
title: "pymongo 依赖库 bson 的 ObjectId 初探"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - pymongo
  - mongodb
  - bson
  - source-code
---

## JSON 与 BSON

mongodb 使用 bson 作为文档内部存储格式, 与我们平常看到的 JSON 查询结果不同. 二者的区别可参见
[JSON and BSON | MongoDB](https://www.mongodb.com/json-and-bson).

简单来讲, BSON 可以理解为 Binary JSON, 主要有两点不同:

- BSON 使用二进制存储, 可读性差
- BSON 的数据类型是 JSON 的超集


而我们在使用 MongoDB 时看到其中的数据与 JSON 最明显的区别就是每个文档中的 `_id` 字段的值,
它的类型就是 BSON 特有的 `ObjectId`.

在 MongoDB 中, 默认 `_id` 作为主键键名, 键值类型为 `ObjectId`. 例:
```js
{
    "_id" : ObjectId("5f9c3200803d95bf660b2d13"),
    "integer_" : 42.0,
}
```
乍看之下, 内部还是一个字符串值, 那么这个 `ObjectId` 类型究竟有什么特别呢?



## bson.ObjectId

我们在 `pip install pymongo` 时, 一并安装的还有 `bson` 包, 用于处理 BSON 数据.

在安装目录 `site-packages/bson` 就可以找到 `objectid.py`

源码见 [objectid.py](https://github.com/mongodb/mongo-python-driver/blob/master/bson/objectid.py)

首先比较直观的几个事实:

- `ObjectId` 直接继承于 `object`

- 定义了 `__slots__`, 所以实例的属性只有 `__id`

- 一个 `ObjectId` 实例(后称为 `oid`) 由 12 个字节 (byte) 组成:
    - 前 4 字节为时间戳
    - 中间 5 字节为随机值
    - 后 3 字节为由一个随机值起始的计数器

因为一个 16 进制数字占据的存储空间为 4 位(bit), 也就是半个字节.
所以 oid 的字符串表示是一个长度为 24 的 16 进制字符串.

也就是说, oid 和 字符串是可以互转的, 但对字符串有限制:

- 长度为 24

- 每项仅限为 `string.hexdigits.lower()`, 也就是 `0123456789abcdefabcdef`


## 作为主键的优势

1. 保证唯一性, 前四字节的时间戳虽时间而增长, 中间五字节的生成和机器有关,
后三字节由某随机数开始自增, 联合保证生成的唯一性.

2. 自带时间戳, 无需再添加一个 `creation_datetime` 的字段去存放当前时间, 可直接取前四字节转换.

3. 可比较性, `ObjectId` 定义了一系列的比较方法, 因为时间戳为前四字节, 至少能根据时间先后进行筛选.


关于 `2` 和 `3`, `ObjectId` 提供了可调用的方法:

```python
from bson.objectid import ObjectId

def test_oid():
    now = datetime.utcnow()
    # 根据当前时间生成的 oid 后 8 字节都为 0
    oid0 = ObjectId.from_datetime(now)

    oid = ObjectId()
    s = str(oid)

    assert ObjectId(s) == oid
    assert oid.generation_time.timestamp() > now.timestamp()
    assert oid > oid0

    # 可通过 datetime 生成 oid 并比较大小筛选数据
    # cursor = col.find({"_id": {"$gte": oid0}})

```

## 总结

我们可以通过 `ObjectId` 提供的方法与 `str`, `datetime` 类型进行互转.

由于 `ObjectId` 的数据组成格式和可比较性, 我们可以将其作为时间戳使用和查询数据.
