---
title: "使用聚合实现 MongoDB 有条件的 distinct"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - mongodb
  - pymongo
---

当我们需要查询文档某个属性的不重复值时, 可以使用 `distinct` 方法, 如:

```js
db.collection.distinct('visit_id')
```

等同于关系数据库的:

```sql
SELECT DISTINCT visit_id FROM collection
```

但是当我们需要在一定的查询条件下执行 `distinct` 时就不支持了, 因为 `find` 后面不能跟 `distinct`.

但是我们可以使用聚合 `aggregate` 曲折地实现这一目的.

假设我们的文档格式如下:

- uid: string, 用户 id
- name: string, 用户名
- email: string, 邮箱
- datetime: Date, 创建时间
- article: array of object 文章列表
  - aid: string, 文章 id
  - title: string, 标题
  - content: string, 内容
  - tags: array of string, 标签


将聚合分为以下步骤:

- `$match` 按条件进行筛选
- `$unwind` 平铺文章列表, 将含有 N 个文章的记录拆分为 N 个文档, article 类型变为 object
- `$group` 用 aid 分组得到 distinct 的 aid
- `$sort` 可选进行排序

得到的查询如下:

```js
db.collection.aggregate([
    {
      "$match": {
          "datetime": {
              "$gte": start,
              "$lte": end
          }
      }
    }, {
      "$unwind": "$article"
    }, {
      "$group": {
          "_id": "$article.aid"
      }
    }, {
      "$sort": {"_id": 1}
    }
  ]
)
```

查询每次返回一个 object, 只有一个属性 `_id`, 获取其内容即可.

关于聚合, 可以查看官方文档的 [简要介绍](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/) 和所有的 [操作列表](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/)

关于聚合操作的练习, 可以使用官方提供的客户端 [MongoDB Compass](https://www.mongodb.com/try/download/compass), 
可以实时的展示聚合每一步的结果, 方便学习与调试, 
还可以直接导出为某个语言(包括 Python Node C# Java)的查询语句.

最后, `distinct` 的函数签名其实为 `function (keyString, query, options) {}`,
是可以传入上文聚合中 `$match` 对应的内容作为 `query` 的值的. 还是要多看文档.

参见 [specify-query-with-distinct](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#specify-query-with-distinct)
