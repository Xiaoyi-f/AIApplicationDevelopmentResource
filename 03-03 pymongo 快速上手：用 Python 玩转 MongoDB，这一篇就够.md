---
title: "pymongo 快速上手：用 Python 玩转 MongoDB，这一篇就够"
status: 已发布
author: XiaoYi
---

# pymongo 快速上手：用 Python 玩转 MongoDB，这一篇就够

> 爬虫抓来的数据、结构多变的文档、用户行为记录……很多场景用 MySQL 存会很别扭，这时候就轮到 **MongoDB + pymongo** 出场了。 今天一篇，带你从「MongoDB 是啥」到「增删改查全搞定」，全程配能直接跑的代码。

## 一、先搞懂：MongoDB 是啥？

一句话：**MongoDB 是一个「文档型」数据库，数据存成一条条像 JSON 一样的「文档」。**

和 MySQL 这种「表格型」对比着看，一下就懂：

|  |  |  |
| --- | --- | --- |
|  | MySQL（关系型） | MongoDB（文档型） |
| 数据库 | database | database |
| 表 | table | **collection（集合）** |
| 一行数据 | row | **document（文档）** |
| 一列 | column | **field（字段）** |
| 结构 | 必须先定义表结构 | **字段随意，每条文档可以不一样** |

**MongoDB 最大的好处就是「灵活」**：你不用提前建表、定字段，想存啥就存啥，每条数据结构还能不一样。所以它特别适合存**爬虫抓来的脏数据、日志、用户资料**这类「结构经常变」的东西。

> 而 **pymongo**，就是 Python 官方推荐的、用来操作 MongoDB 的库。

---

## 二、环境准备

```
uv pip install pymongo
```

MongoDB 可以本地装、Docker 跑，或直接用云服务。连的时候要一个地址，默认本地是：

```
mongodb://localhost:27017
```

---

## 三、连接：三步拿到「集合」

```
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017")
db = client["db"]
collection = db["collection"]
```

> 记住这三步：**client（服务）→ db（库）→ collection（集合）**。后面所有操作都在 `users` 这个集合上做。

---

## 四、增（Create）：插入文档

```
# 增
collection.insert_one({"key": "value"})

collection.insert_many([
    {"key": "value"},
    {"other_key": "other_value"}
])
```

> 注意：你没给 `_id` 时，MongoDB 会自动生成一个唯一主键 `_id`，不用自己管。

---

## 五、查（Read）：找到你要的数据

### 查一条 / 查全部

```
one = collection.find_one({"key": "value"})
all = list(collection.find({"city": "北京"}))
```

### 条件查询：比较运算符

```
collection.find({"key": {"$gt": 18}}) # $gte $lte $lt $ne
```

> 常用运算符速记：`$gt` 大于、`$gte` 大于等于、`$lt` 小于、`$lte` 小于等于、`$ne` 不等于。

### 正则查询（模糊匹配）

```
collection.find({"key": {"$regex": "^."}})
```

---

## 六、改（Update）：更新文档

```
collection.update_one({"key": "value"}, {"$set": {"other_key": "other_value"}})
collection.update_many({"key": "value"}, {"$set": {"other_key": "other_value"}})
collection.update_one({"key": "value"}, {"$inc": {"age": 1}}) # age + 1
```

> 关键：**改字段一定要用 `$set`**，不然会把整条文档替换掉，别的字段就没了。`$inc` 用来做数字自增/自减。

---

## 七、删（Delete）：删除文档

```
collection.delete_one({"key": "value"})
collection.delete_many({"key": {"$lt": 18}})
```

> ⚠️ 删除不可逆，`delete_many` 删之前一定要确认条件范围。

---

## 八、进阶：排序、分页、索引、聚合

```
# 排序
collection.find().sort("key", -1) # -1 表示降序, 1 表示升序

# 分页
collection.find().skip(10).limit(10) # 跳过前10条再取10条

# 建索引
collection.create_index("key")

# 聚合
pipeline = [
    {"$group": {"_id": "$key", "count": {"$sum": 1}}} # 1 表示遇到一条就加1
]
res = list(collection.aggregate(pipeline))
```

---

## 九、什么时候用 MongoDB，什么时候用 MySQL？（一句话决策）

- **结构固定、要事务、要关联查询** → 用 **MySQL**

- **结构多变、存文档/日志/爬虫数据、要高扩展** → 用 **MongoDB**

> 别纠结「谁更牛」，它们不是对手，是搭档。真实项目里经常**两个一起用**——MySQL 存核心业务，MongoDB 存灵活数据。

---

我是小翊，一个 19 岁把前端、后端、爬虫、数据库、AI 全扛下来的全干工程师。这个系列我会一篇篇写下去，下一篇带你用 pymongo + 爬虫，做一个「数据采集 + 存储」的小项目。

`Let's use AI to create a better world.`
