---
title: "Neo4j 图数据库：从 Cypher 入门到 Louvain 社区发现"
status: 已发布
author: XiaoYi
---

# Neo4j 图数据库：从 Cypher 入门到 Louvain 社区发现

> 图数据库是我做「技能图谱 / 人岗匹配」时的主力。这篇是我从真实项目里沉淀的完整笔记，从 Cypher 语法一路讲到社区聚类算法。 Let's use AI to create a better world。

## 一、Neo4j 到底是个啥？

一句话：**Neo4j 是一种「图数据库」，专门用来存「实体和实体之间的关系」。**

它基于**图论**，跟 MySQL 那种「表格」完全不是一个思路：

|  |  |  |
| --- | --- | --- |
|  | MySQL（关系库） | Neo4j（图库） |
| 存什么 | 一张张表 | 一个个「点 + 线」 |
| 表达关系 | 靠 JOIN 连表（费劲） | 关系就是一条线（天生会） |
| 擅长 | 事务、强一致 | 关系查询、路径、社交网络 |

> 比喻：MySQL 是「账本」，一行行记；Neo4j 是「关系图」，人和人的连线画在纸上，一眼就能看穿「谁认识谁、谁和谁有关」。

### 什么时候必须用图库？

当你遇到这种需求——「**查 A 和 B 之间隔了几层关系**」「**把一群人按关系聚成几个圈子**」，MySQL 写几十行 JOIN 还慢，Neo4j 一条 Cypher 就搞定。

这就是为什么「技能图谱」「人岗匹配」「社交推荐」「知识图谱」这些场景，都用图数据库。

### 两个重要概念

- **LPG（标签属性图）**：Neo4j 用的模型，每个节点/关系都带标签和属性

- **RDF（资源描述框架）**：另一种图模型，W3C 标准，偏知识图谱语义网

> 日常用 Neo4j，你打交道的就是 **LPG**，记住「节点、标签、属性、关系」这四个词就够了。

---

## 二、环境与访问（几个关键点）

- Neo4j 用 **Java 和 Scala** 实现，**依赖 Java 环境**

- **5.x 系列**：5.26 是 LTS（长期支持版），5.x 用 **Java 17**；再往后的版本需要 **Java 21+**

- 装好后，访问 **数据浏览器**（浏览器里写 Cypher 的图形界面）：`http://localhost:7474`

- 命令行工具 `cypher-shell`：

```
cypher-shell -u neo4j -p password -d db_name -f xxx.cypher
```

> 看到 `HTTP enabled on localhost:7474`，就说明数据浏览器能用了。

---

## 三、Cypher 语法基础

**Cypher** 是 Neo4j 的查询语言，可以理解成「图版的 SQL」——但它是**用画画的方式**描述数据。

### 3.1 核心语法：节点和关系

```
(变量:标签 {属性键: 属性值})-[变量:关系类型]->(变量:标签 {属性键: 属性值})
```

拆开看：

- **节点**用圆括号 `()`：`(n:Person {name: "小明"})` = 一个标签为 Person、name 是小明的节点

- **关系**用方括号 + 箭头 `-[]->`：`-[r:KNOWS]->` = 一条「认识」的关系，方向从左到右

> 记忆：**圆括号装人，方括号装关系，箭头指方向。**

### 3.2 基本类型（支持嵌套）

字符串、整数、浮点数、布尔、列表、字典。字典和列表还能嵌套，和 Python 很像。

---

## 四、增删改查（Cypher 的核心）

### 4.1 查：MATCH

```
// 查一个具体人
MATCH (node:Person {name: "小明"}) RETURN node

# 查所有人
MATCH (n) OPTIONAL MATCH (n)-[r]->(m) RETURN n, r, m
```

`OPTIONAL MATCH` 是「可选的匹配」——即使某节点**没有**对应关系，也把它查出来，关系字段填 null。

### 4.2 条件：WHERE

```
// 年龄大于 18 且 名字包含 聂
MATCH (n) WHERE n.age > 18 AND n.name CONTAINS "聂" RETURN n

// 正则
MATCH (n) WHERE n.name =~ "^聂.*" RETURN n
```

- `CONTAINS`：包含

- `=~`：正则匹配

### 4.3 增：CREATE / MERGE

```
// 创建一个结点
CREATE (n:Person {name: "聂宝"})

// MERGE 存在就匹配 不存在就创建
MERGE (n:Person {name: "聂宝"}) RETURN n
```

> 记牢：**CREATE 每次都会新建（可能重复），MERGE 会先找、找不到才建（幂等）**。日常写数据优先用 MERGE。

### 4.4 改：SET

```
// 赋值 / 追加属性
MATCH (n) SET n.age = 20

// MERGE + 条件设置: 新建时设一套, 已存在时设另一套
MERGE (n:Person)
ON CREATE SET n += {from: "文档"} // 新建时执行
ON MATCH SET n += {update_time: date()} // 已存在时执行
```

### 4.5 删：DELETE / REMOVE

```
// 删节点 + 它的所有关系 (DETACH 必须先切断关系才能删除节点 而 DETACH DELETE 可直接删)
MATCH (n) DETACH DELETE n

// 只删标签 / 只删属性
MATCH (n) REMOVE n:Tag // 删除标签
MATCH (n) REMOVE n.attr // 删除属性
```

> ⚠️ Neo4j 里**有关系的节点不能直接删**，必须 `DETACH DELETE` 先断开关系再删。

---

## 五、约束和索引

### 5.1 唯一约束（防止重复）

```
CREATE CONSTRAINT person_name_unique IF NOT EXISTS
FOR (n:Person)
REQUIRE n.name IS UNIQUE;

DROP CONSTRAINT person_name_unique IF EXISTS;
```

### 5.2 索引（加速查询）

```
CREATE INDEX person_name_index IF NOT EXISTS
FOR (n:Person)
ON (n.name);

SHOW INDEXES; -- 查看所有索引

DROP INDEX person_name_index IF EXISTS;
```

> 和 MySQL 一样：**约束保证数据不重复，索引保证查询快。**

---

## 六、库、事务

### 6.1 库管理

```
CREATE DATABASE db_name;
:USE db_name;
DROP DATABASE db_name;
```

### 6.2 事务

```
:BEGIN
// 这里写若干条 CQL 语句
:ROLLBACK
:COMMIT
```

---

## 七、导入 CSV 数据

```
// CSV 必须是 UTF-8 无 BOM 头 读出来数据默认为字符串类型
LOAD CSV WITH HEADERS FROM "file://data.csv" AS row
MERGE (n:Person {name: row.name})
```

> 两个坑：① CSV 要 **UTF-8 且无 BOM**，否则中文乱码；② 读出来的**全是字符串**，要数字得自己 `toInteger()` 转换。

---

## 八、Docker 部署 + GDS 配置

```
neo4j:
  image: neo4j:5
  environment:
    NEO4J_AUTH: neo4j/neo4j
    NEO4J_dbms_security_procedures_allowlist: gds.*
    NEO4J_dbms_security_procedures_unrestricted: gds.*
  ports:
    - "7474:7474" # 浏览器
    - "7687:7687" # Bolt 协议 (程序)
```

> 要用图算法（Louvain 等），必须把 `gds.*` 加进这两个白名单，否则算法过程调不动。

---

## 九、Louvain 社区发现（图数据科学）

这是图数据库真正拉开差距的能力：**自动把一张关系网聚成几个「社区」**（比如：把技能图谱里的人按技能聚成不同圈子）。

```
# uv pip install graphdatascience

from graphdatascience import GraphDataScience

gds = GraphDataScience("bolt://localhost:7687", auth=("neo4j", "password"))

G, res = gds.graph.project("graph_name", "*", "*", {"orientation": "UNDIRECTED"})
# res -> res.nodeCount res.relationshipCount

ret = gds.louvain.write(G, writeProperty="community_id")
# ret["communityCount"] 社区数量 / ret["modularity"] 模块度, 越接近 1 聚类效果越好

G.drop() # 释放内存投影图
```

几个实战要点：

- **图投影**是必须的：算法不能直接在磁盘跑，要先投影到内存

- **内存预算**：1000 万条数据，至少准备 **6GB 空闲内存**

- **模块度（modularity）**：越接近 1，说明「圈内关系密、圈间关系疏」，聚类效果越好

- 跑完记得 `G.drop()`，否则内存一直被占着

> 这一节就是你做「技能图谱 → 自动聚类」的核心武器，也是图数据库相对其他方案最硬核的差异点。

---

## 十、Python 驱动（程序里操作 Neo4j）

```
from neo4j import GraphDatabase

URI = "neo4j://localhost:7687"
AUTH = ("neo4j", "password")
DB_NAME = "neo4j"

def operate():
    # records 查询结果列表 / summary.query 执行的语句/summary.database 数据库名 / keys 字段
    with GraphDatabase.driver(URI, auth=AUTH) as driver:
        records, summary, keys = driver.executor_query(
            """
            MATCH (n:Person {name: $name}) RETURN n.name AS name
            """
            name="XiaoYi",
            database=DB_NAME
        )

        def read_tx(tx):
            return tx.run("MATCH (n) RETURN n LIMIT 5").data()

        def write_tx(tx):
            return tx.run("MERGE (n:Person {name: $name})", name="测试").data()

        with driver.session(database=DB_NAME) as session:
            read_data = session.execute_read(read_tx)
            write_data = session.execute_write(write_tx)
```

几个要点：

- **推荐 `execute_query`**：一次拿齐「结果 + 统计 + 字段名」，比手动管事务省心

- **读写分离**：读用 `execute_read`，写用 `execute_write`，Neo4j 会自动处理事务和重试

- `$name` 是**参数化查询**，防止 Cypher 注入，别用字符串拼接

---

## 总结：一页速查

|  |  |
| --- | --- |
| 操作 | Cypher / 代码 |
| 查节点 | `MATCH (n:Person {name:"小明"}) RETURN n` |
| 查关系（可选） | `MATCH (n) OPTIONAL MATCH (n)-[r]->(m) RETURN n,r,m` |
| 条件 | `WHERE n.age > 18 AND n.name CONTAINS "聂"` |
| 正则 | `WHERE n.name =~ "^聂.*"` |
| 创建（防重复） | `MERGE (n:Person {name:"聂宝"})` |
| 改属性 | `SET n.age = 20` |
| 删节点+关系 | `MATCH (n) DETACH DELETE n` |
| 唯一约束 | `CREATE CONSTRAINT ... REQUIRE n.name IS UNIQUE` |
| 建索引 | `CREATE INDEX ... ON (n.name)` |
| 导入 CSV | `LOAD CSV WITH HEADERS FROM "file://x.csv" AS row` |
| 社区发现 | `gds.louvain.write(G, writeProperty="community_id")` |

> 一句话记牢：**Neo4j 用「点 + 线」存关系，Cypher 用画画的方式查询，MERGE 防重复、DETACH DELETE 删干净，Louvain 帮你把关系网聚成社区。**
