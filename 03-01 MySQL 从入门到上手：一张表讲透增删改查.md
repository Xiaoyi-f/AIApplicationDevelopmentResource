---
title: "MySQL 从入门到上手：一张表讲透增删改查"
status: 已发布
author: XiaoYi
---

# MySQL 从入门到上手：一张表讲透增删改查

> 今天把 MySQL 从「仓库是什么」到「增删改查全会」，用最土的话一次讲透。 Let's use AI to create a better world。

## 一、MySQL 到底是个啥？

一句话：**MySQL 是一个「放表格的大仓库」。**

你平时记东西用什么？Excel 表格对吧。MySQL 就是**把无数张 Excel 表格，整整齐齐存进一个仓库里**，让你能：

- 快速往表格里加数据、删数据、改数据、查数据

- 几百万行数据，也能 0.1 秒翻出来

它最大的特点是**关系型**——表格和表格之间能「扯上关系」。比如「用户表」和「订单表」，靠一个 `user_id` 就能连起来。

先记住这套对应关系，后面全用得上：

|  |  |
| --- | --- |
| 生活里的说法 | MySQL 里的叫法 |
| 一个仓库 | 数据库 database |
| 仓库里的一张表格 | 表 table |
| 表格的一行 | 记录 row |
| 表格的一列 | 字段 field / column |
| 每行唯一的编号 | 主键 primary key |

> 记忆：**一个仓库（database）里，摆着一张张表格（table），表格里一行行（row）一列列（column）。**

---

## 二、为什么你必须学它？

- 它是全世界**最主流**的关系型数据库，大厂、外包、接单、做网站，几乎都绕不开

- 你会了它，学 PostgreSQL、SQLite 等其他 SQL 库**几乎是免费的**（语法九成一样）

- 你后面用 Spring Boot + MyBatis-Plus，底层生成的也是这些 SQL，**懂 SQL 才看得懂日志、调得了性能**

---

## 三、环境准备（30 秒上手）

装好之后，用命令行登录：

```
# 修改root密码
alter user 'root'@'localhost' identified by 'new_password';
exit;

mysql -uroot -p
# 输入密码

# 使用建表基本模板
SET NAMES utf8mb4;
USE `xxxdb`;
SOURCE xxx.sql;
```

输入密码，出现 `mysql>` 就进去了。下面所有代码都在这个黑框里敲。

---

## 四、第一步：建仓库 + 建表

### 4.1 建一个数据库（仓库）

```
create database db;
USE db;
```

### 4.2 建一张表（表格）

假设我们要存「用户」，先想好每行要记哪些东西：

```
create table table_name (
    id         BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    foreign_id BIGINT UNSIGNED NOT NULL,
    name       VARCHAR(50) NULL,
    enable     TINYINT(1) NOT NULL DEFAULT 0,
    city       VARCHAR(50),
    age        INT,
    salary     DECIMAL(10, 2), -- 最多10位 小数2位
    descript   TEXT NULL    COMMENT '描述',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    -- KEY `idx_resume`  (`name`) UNIQUE KEY 和 普通 KEY 不可同时给同一个字段
    PRIMARY KEY          (`id`),
    UNIQUE KEY `uk_name` (`name`),
    CONSTRAINT `fk_table_other` FOREIGN KEY (`foreign_id`)
        REFERENCES `other_table`(`id`)
        ON DELETE CASCADE
        ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='标注释';
```

几个字段类型先记这几个够用：

|  |  |  |
| --- | --- | --- |
| 类型 | 用途 | 例子 |
| `INT` | 整数 | 年龄、数量 |
| `VARCHAR(n)` | 变长字符串，最多 n 个字符 | 名字、城市 |
| `TEXT` | 长文本 | 简介、文章内容 |
| `DECIMAL(10,2)` | 小数（钱） | 工资、价格 |
| `DATETIME` | 日期时间 | 下单时间 |
| TINYINT | 常用作boolean | 可行否 |

---

## 五、增删改查（核心中的核心）

记住四个单词，MySQL 就入门了：

> **增 Insert、删 Delete、改 Update、查 Select**（合起来叫 CRUD）

### 5.1 增：INSERT

```
-- 插入一条
insert into table_name (name, city, age, salary) values ('XiaoYi', '深圳', 19, 500000);

-- 插入多条
insert into table_name (name, city, age, salary) values
    ('小明', '北京', 22, 150000),
    ('小红', '上海', 25, 180000),
    ('小刚', '北京', 28, 250000);
```

### 5.2 查：SELECT（用的最多）

```
select * from table_name;
select name, city from table_name;
select * from table_name where city = '北京';
```

### 5.3 改：UPDATE（⚠️ 一定带 WHERE）

```
update table_name set salary = 800000 where name = 'XiaoYi';
update table_name set age = 20, city = '杭州' where name = 'XiaoYi';
```

> ⚠️ **忘了写 WHERE，整张表都会被你改掉！** 这是新手最痛的血泪教训。

### 5.4 删：DELETE（⚠️ 同样一定带 WHERE）

```
delete from table_name where name = 'XiaoYi';
```

> ⚠️ 同样，`DELETE FROM users;`（不带 WHERE）会**清空整张表**。

---

## 六、查询进阶：真正拉开差距的地方

### 6.1 条件 WHERE 全家桶

```
select * from table_name age >= 22;
select * from table_name city != '北京';
select * from table_name where age between 20 and 25;
select * from table_name where city in ('北京', '上海');
select * from table_name where like 'Xiao%';
select * from table_name where city = '北京' and age > 20; -- not and or
```

### 6.2 排序 ORDER BY

```
select * from table_name order by salary desc limit step, size; -- asc 升序, desc 降序
```

### 6.3 分组 + 聚合（数个数、求和）

这就是你上次问 pymongo 那个「分组数个数」的 **SQL 版本**，逻辑一模一样：

```
select city, count(*) as count from table_name group by city;
select city, avg(salary) as avg_salary from table_name group by city;
-- min() max() sum()
```

对应关系，一眼看穿：

|  |  |  |
| --- | --- | --- |
| 目的 | pymongo | MySQL |
| 分组 | `$group` | `GROUP BY` |
| 按哪个字段分 | `_id: "$city"` | `GROUP BY city` |
| 数个数 | `{"$sum": 1}` | `COUNT(*)` |

常用聚合函数：`COUNT(*)` 数个数、`SUM(x)` 求和、`AVG(x)` 平均、`MAX(x)` 最大、`MIN(x)` 最小。

### 6.4 分组后筛选 HAVING

`WHERE` 是「分组前筛」，`HAVING` 是「分组后筛」：

```
select city, count(*) as count
from table_name
group by city
having count(*) > 1;
-- where 分组前筛选 / having 分组后筛选
```

---

## 七、两张表连起来：JOIN

这是关系型数据库的**灵魂**。假设还有一张订单表：

```
select users.name, orders.amount
from orders
inner join users on orders.user_id = users.id;
-- inner join 内连接
-- left join 左连接
```

|  |  |
| --- | --- |
| JOIN 类型 | 一句话 |
| `INNER JOIN` | 两边都有才算 |
| `LEFT JOIN` | 左边全保留，右边对不上就填 NULL |

---

## 八、索引：为什么它这么快

没索引时，查一条数据要**一行行翻**（全表扫描），几百万行就慢死了。

加个索引，就像给书加**目录**，直接翻到那一页：

```
-- 索引
create index idx_city on users (city);
```

一句话：**经常用来查的字段（WHERE 后面的），给它建索引。**

## **九、事务**

```
start transaction;
savepoint sp;
rollback to sp;
rollback;
commit;
```

---

## 十、和你的 Spring Boot + MyBatis-Plus 串起来

你项目里写 Java，大概率是这么用：

- **MyBatis-Plus** 帮你把 `User` 实体类自动映射成 `users` 表，`userMapper.selectById(1)` 底层就是 `SELECT * FROM users WHERE id = 1`

- 复杂查询、多表 JOIN、聚合统计，还是要**自己手写 SQL**（在 Mapper 里写 `@Select(...)` 或 XML）

所以：**MyBatis-Plus 帮你省掉重复的 CRUD，但底层 SQL 你必须懂**——不然日志里那一行行 SQL 你看不懂，慢查询也调不了。

## 补充：备份与恢复

```
mysqldump -h -uroot -p db_name [--database db_names] [--all-database] > save.sql
mysql -uroot -p db_name < save.sql
mysql -uroot -p < save.sql

-- 定时备份
-- 创建 /data/mysql_backup.sh 脚本
#!/bin/bash
CONTAINER_NAME="mysql8"
MYSQL_USER="root"
MYSQL_PASS="password"
BACKUP_DIR="/data/mysql_backup"
RETENTION=7 # 保留7天备份

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/db_backup_${DATE}.sql.gz"

# 创建目录
mkdir -p ${BACKUP_DIR}

# 核心命令
docker exec ${CONTAINER_NAME} mysqldump \
    -u${MYSQL_USER} -p${MYSQL_PASS} \
    --single-transaction # 适配Innodb引擎
# --single-transaction 热备份
# - 备份期间，不能有 `ALTER TABLE` / `DROP` / `RENAME` 这类 DDL，会破坏快照，导致备份失败
# - 不适合超大表、长时间 dump 快照会持续占用事务
| gzip > ${BACKUP_FILE}

# 删除七天前旧备份
find ${BACKUP_DIR} -name "db_backup_*.sql.gz" -mtime +${RETENTION} -delete
echo "[$(date '+%Y-%m-%d %H:%M:%S')] 备份完成: ${BACKUP_FILE}"

-- chmod +x /data/mysql_backup.sh
-- /data/mysql_backup.sh 执行
crontab -e
0 2 * * * /data/mysql_backup.sh >> /data/mysql_backup/backup.log 2>&1

crontab -l

-- 解压执行恢复
mysql gunzip -c /data/mysql_backup/db_backup_xxx.sql.gz | docker exec -i mysql8 mysql -uroot -ppassword
```

---

##

## 总结：一页速查

|  |  |
| --- | --- |
| 操作 | SQL |
| 建库 | `CREATE DATABASE x;` |
| 建表 | `CREATE TABLE x (...);` |
| 增 | `INSERT INTO x (...) VALUES (...);` |
| 查 | `SELECT * FROM x WHERE ...;` |
| 改 | `UPDATE x SET ... WHERE ...;` |
| 删 | `DELETE FROM x WHERE ...;` |
| 排序 | `ORDER BY 字段 DESC` |
| 分组 | `GROUP BY 字段` |
| 连表 | `JOIN y ON x.id = y.x_id` |
| 建索引 | `CREATE INDEX i ON x(字段);` |

> 记住四个单词就入门：**Insert、Delete、Update、Select**。剩下的 JOIN、索引、事务，是用起来之后再逐个攻克的。

---
