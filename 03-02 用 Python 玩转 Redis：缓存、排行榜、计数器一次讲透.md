---
title: "用 Python 玩转 Redis：缓存、排行榜、计数器一次讲透"
status: 已发布
author: XiaoYi
---

# 用 Python 玩转 Redis：缓存、排行榜、计数器一次讲透

> 今天带你用 Python 把 Redis 从「是什么」到「缓存 + 排行榜」全跑通。 Let's use AI to create a better world。

## 一、Redis 到底是个啥？

一句话：**Redis 是一个「超级快的内存仓库」，数据都存在内存里，不落硬盘（也可以定期落盘）。**

MySQL 像「硬盘仓库」——容量大、能永久存，但每次读写要转磁盘，慢。 Redis 像「抽屉」——伸手就能拿到，**快得离谱**（每秒能处理十万次读写），但内存贵、容量小。

所以实际项目里，它俩是搭档：

- **MySQL** 存「真数据」，永久保存

- **Redis** 存「热数据」，给 MySQL 挡在最前面加速

对应关系先记住：

|  |  |
| --- | --- |
| 生活里的说法 | Redis 的叫法 |
| 一个钥匙配一个锁 | 键值对 key-value |
| 值有不同类型 | 5 种数据类型 |
| 给钥匙设个有效期 | 过期时间 TTL |

---

## 二、Redis 用来干嘛？（都是赚钱场景）

1. **缓存**：查过的东西先放 Redis，下次秒回，不反复打数据库

1. **排行榜**：点赞榜、销量榜，用有序集合天然支持

1. **计数器**：浏览量、点赞数，`incr` 一下 +1

1. **分布式锁**：多个程序抢同一份资源，用 Redis 锁住（你 Spring Boot 里用 Redisson 干的就是这个）

1. **消息队列**：列表 push/pop 当队列用

---

## 三、环境准备

```
pip install redis
```

装好后，本地起一个 Redis 服务（Docker 最快）：

```
docker run -d --name redis_container -p 6379:6379 redis
```

默认端口是 **6379**。

---

## 四、连接：万事开头三步

```
import redis

r = redis.Redis(
    host="localhost",
    port=6379,
    db=0,
    decode_responses=True
)

r.ping() # 测试联通 联通成功返回 True
```

> ⚠️ `decode_responses=True` 一定要加。不加的话，你 `get` 出来的是 `b'xxx'`（字节串），中文会变成乱码。

---

## 五、5 种数据类型（重点）

Redis 的「值」有 5 种类型，每种对应一个场景：

|  |  |  |
| --- | --- | --- |
| 类型 | 样子 | 典型场景 |
| 字符串 String | 一个 key 存一个值 | 缓存、计数器 |
| 哈希 Hash | 一个 key 存一堆字段 | 存对象（用户信息） |
| 列表 List | 有序的一串 | 队列、消息流 |
| 集合 Set | 无序、不重复 | 去重、共同好友 |
| 有序集合 ZSet | 有序、不重复、带分数 | 排行榜 |

---

## 六、字符串 String（最常用）

```
# 存
r.set("name", "XiaoYi")

# 取
name = r.get("name")

# 删
r.delete("name")

# 计数器
r.incrby("view_count", -1) # 正负数均可
```

### 缓存的核心：给 key 设过期时间

缓存不能永久留着（会占满内存、数据也旧了），所以要设有效期：

```
# 存 30秒 自动消失
r.setex("hot_data", 30, "hot_value")

# 先存再设过期
r.set("key", "value")
r.expire("key", 30)

# 看剩余时间
r.ttl("key")
```

> 缓存三板斧：**set 存 → get 取 → setex/expire 设过期**。

---

## 七、哈希 Hash（存对象）

一个 key 里存多个字段，适合存「一个用户的信息」：

```
# 一次性存多个字段
r.hset("user:1", mapping={
    "name": "小翊",
    "city": "深圳",
    "age": "19"
})

# 取单个字段值
r.hget("user:1", "name") # 小翊

# 取全部
r.hgetall("user:1") # {"name": "小翊", "city": "深圳", "age": "19"}

# 删字段
r.hdel("user:1", "age")
```

---

## 八、列表 List（当队列用）

```
# 左边塞
r.lpush("queue", "任务1", "任务2")

# 右边塞
r.rpush("queue", "任务3")

# 从左起取全部
r.lrange("queue", 0, -1)

# 左边弹出一个
r.lpop("queue")
```

> 用 `rpush` 生产 + `lpop` 消费，就是一个最简单的消息队列。

---

## 九、集合 Set（去重 + 交集）

```
# 加成员
r.sadd("tags", "python", "redis")

# 看所有成员
r.smembers("tags")

# 判断存在与否
r.sismember("tags", "python") # True

# 两个集合的交集
r.sinter("set1", "set2")

# 两个集合的并集
r.sunion("set1", "set2")

# 两个集合的差集
r.sdiff("set1", "set2") # 前边集合减去后边集合重叠部分的最终结果
```

---

## 十、有序集合 ZSet（排行榜神器）

这是 Redis 最值钱的类型：**每个成员带一个分数，自动按分数排序**。

```
r.zadd("rank", {
    "小翊": 100,
    "小红": 80,
    "小明": 95
})

# 按照分数从低到高取
r.zrange("rank", 0, -1, withscores=True)

# 按照分数从高到低取 从 0 索引 到 末尾
r.zrevrange("rank", 0, -1, withscores=True)
# [('小翊', 100.0), ('小红', 95.0), ('小明', 80.0)]

# 只取前两名
z.zrevrange("rank", 0, 1, withscores=True)

# 给某人加分
r.zincrby("rank", 10, "小明")

# 查某人分数 / 排名
r.zscore("rank", "小翊")
```

> 排行榜三件套：**zadd 存分数 → zrevrange 从高到低取 → zincrby 加分**。

---

## 十一、实战：给 MySQL 加一层缓存

这是真实项目里 Redis 的**最大用途**。逻辑是：

> ```
> 查数据：先问 Redis → 有就直接返回（快）
>                       → 没有再去查 MySQL，查完顺手存进 Redis（下次就快了）
> ```

```
# uv pip install sqlmodel pymysql

import redis
from sqlmodel import SQLModel, Field, Session, create_engine

class User(SQLModel, table=True):

    id: int | None = Field(default=None, primary_key=True)
    name: str
    city: str | None = None
    age: int | None = None

engine = create_engine("mysql+pymysql://root:password@localhost/table_name")

# 首次运行建表
SQLModel.metadata.create_all(engine)

r = redis.Redis(host="localhost", port=6379, db=0, decode_responses=True)

def get_user(user_id):
    cached = r.get(f"user:{user_id}")
    if cached:
        return cached

    with Session(engine) as session:
        user = session.get(User, user_id)

    if user:
        r.setex(f"user:{user_id}", 60, user.name)
        return user.name
    return None
```

---

## 十二、和你的 Spring Boot + Redisson 串起来

你在 Java 那边用 Redis，大概率是这两件事：

1. **缓存**：`@Cacheable` 注解或手动 `RedisTemplate`，和上面 Python 的 `set/get` 一模一样

1. **分布式锁**：用 Redisson 的 `RLock`，防止多个请求同时改同一份数据

Python 这边就用 `redis-py`。**底层命令是相通的**——`SET/GET/EXPIRE/INCR` 这些单词，Java 和 Python 里完全一样，你学会 Python 这套，Java 那套秒懂。

---

## 总结：一页速查

|  |  |
| --- | --- |
| 场景 | Python 代码 |
| 存 / 取 | `r.set(k, v)` / `r.get(k)` |
| 带过期存 | `r.setex(k, 秒, v)` |
| 计数器 | `r.incr(k)` |
| 存对象 | `r.hset(k, mapping={...})` |
| 队列 | `r.rpush(k, v)` / `r.lpop(k)` |
| 去重 | `r.sadd(k, v)` / `r.smembers(k)` |
| 排行榜 | `r.zadd(k, {...})` / `r.zrevrange(k, 0, -1, withscores=True)` |

> 一句话记牢：**Redis = 快，所以当缓存；ZSet = 自带排序，所以做排行榜。** 核心就这两件事，剩下的遇到再查。
