---
title: "使用 pymongo 编写兼容 mongoexport 的脚本"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - script
  - mongodb
  - pymongo
  - python
---

mongodb 提供了两个数据导出工具:

- `mongodump`: 导出数据为 `.bson` 文件, 可使用 `mongorestore` 重新导入
- `mongoexport`: 导出数据为 `.json` 或 `.csv` 文件, 可使用 `mongoimport` 重新导入.

后者导出的文件可读性更好, 默认导出的 json 格式在数据传输过程中被广泛使用.

`pymongo` 是 `mongodb` 的 python 客户端程序, 也可以称之为驱动(`driver`).

mongodb 支持的数据类型如下:

```js
{
    "_id" : ObjectId("5f9c321d803d95bf660b2d14"),
    "integer_" : 42.0,
    "string_" : "something",
    "bool_" : true,
    "double_" : 3.14,
    "array_" : [ 
        42.0, 
        "something", 
        true, 
        3.14
    ],
    "object_" : {
        "key1" : "value1",
        "key2" : "value2"
    },
    "null_" : null,
    "date_" : ISODate("2020-10-30T15:32:45.571Z"),
    "other_data_types" : [ 
        "timestamp", 
        "binary data", 
        "regular expression", 
        "code"
    ],
    "ref" : "https://www.w3schools.in/mongodb/data-types/"
}
```

其中有些类型是没办法直接序列化的, 如 ObjectId 和 ISODate(或 Date 类型).
所以在序列化后需要指定规则.

我们先看一下 mongoexport 导出的数据:

```shell
mongoexport -d test -c test -o test.json
cat test.json
# {"_id":{"$oid":"5f9c321d803d95bf660b2d14"},"integer_":42.0,"string_":"something","bool_":true,"double_":3.14,"array_":[42.0,"something",true,3.14],"object_":{"key1":"value1","key2":"value2"},"null_":null,"date_":{"$date":"2020-10-30T15:32:45.571Z"},"other_data_types":["timestamp","binary data","regular expression","code"],"ref":"https://www.w3schools.in/mongodb/data-types/"}
```

可以看到:
- `ObjectId("value")` 被转为了 `{"$oid": "value"}`
- `ISODate("value")` 被转为了 `{"$date": "value"}`


因此我们只需要将生成的数据转为对应格式即可.

在 python 中, `ObjectId` 对应 `bson.ObjectId`, `Date/ISODate` 对应
`datetime.datetime` 类型.

python 自带模块 `json` 的 `dumps` 方法仅支持序列化基本类型, 即
`str, int, float, bool, None` 以及元素为基本类型的 `list, dict`.

但是有参数预留了补丁, 可以让我们在调用时手动支持其他类型.

我们先看一下 `dumps` 方法的签名:

```python
def dumps(obj, *, skipkeys=False, ensure_ascii=True, check_circular=True,
        allow_nan=True, cls=None, indent=None, separators=None,
        default=None, sort_keys=False, **kw):
    pass
```

其中 `default` 参数就是我们的补丁, 此参数的文档说明:
> `default(obj)` is a function that should return a serializable version
> of obj or raise TypeError. The default simply raises TypeError.

也就是说我们可以直接编写一个函数 default:

```python
def dump_default(o):
    """ObjectId 和 Date 的序列化"""
    if isinstance(o, ObjectId):
        return {"$oid": str(o)}
    if isinstance(o, datetime):
        return {"$date": str(o.isoformat())[:-3] + 'Z'}
    raise TypeError
```

然后编写一个自定义的 `dump` 函数:

```python
def dump_line(dic):
    """生成单行"""
    return json.dumps(
      dic,                    # 需要序列化的对象
      ensure_ascii=False,     # 返回可包含非 ascii 字符
      default=dump_default,   # 自定义补丁
      separators=(',', ':')   # 分隔符, 默认为 `(', ', ': ')`, 去除空格更为紧凑
    ) + '\n'

```

就可以得到兼容 `mongoexport` 命令的数据格式了:

```python
cursor = col.find()

with open(filename, 'w') as f:
    for c in cursor:
        f.write(dump_line(c))
```
