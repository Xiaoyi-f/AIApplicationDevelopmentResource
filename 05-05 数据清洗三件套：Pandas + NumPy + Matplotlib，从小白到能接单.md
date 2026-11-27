---
title: "数据清洗三件套：Pandas + NumPy + Matplotlib，从小白到能接单"
status: 已发布
author: XiaoYi
---

# 数据清洗三件套：Pandas + NumPy + Matplotlib，从小白到能接单

不管你爬数据、做分析、还是喂 AI 知识库，都绕不开一件事——**数据清洗**。 今天用一篇，把 Python 数据三件套讲透：`Pandas`（表格神器）、`NumPy`（算力引擎）、`Matplotlib`（数据照相机）。 全程配能直接跑起来的代码，收藏这一篇，够你用到接单。

---

## 一、先搞懂：数据清洗到底在「洗」什么？

你拿到手的数据，十有八九是**脏的**：

- 有缺失（该有的值没有）

- 有重复（同一条数据出现好几次）

- 有异常（年龄写成 500 岁、价格是负数）

- 类型乱（数字存成了文本 `"123"`）

**数据清洗 = 把这些「脏菜」洗成「能下锅的菜」。** 不洗就用，分析结果全是错的——垃圾进，垃圾出。

> 一句话记住：**数据质量决定结论质量。清洗这步，占了数据分析 70% 的工作量。**

---

## 二、三件套各是干嘛的（一句话定位）

|  |  |  |
| --- | --- | --- |
| 工具 | 一句话定位 | 打个比方 |
| **Pandas** | 处理表格数据 | 洗菜盆，清洗的主力 |
| **NumPy** | 底层数值计算 | 菜刀，又硬又快 |
| **Matplotlib** | 画图看数据 | 照相机，看清数据长啥样 |

Pandas 是主角（90% 的时间都在用它），NumPy 是它的底座，Matplotlib 帮你「看见」数据。三兄弟一起学，效率最高。

**先装环境：**

```
pip install pandas numpy matplotlib
```

---

## 三、Pandas：数据清洗的主力

### 1. 读数据 + 先「看三眼」

```
import pandas as pd

df = pd.read_csv("data.csv")
# pd.read_excel("data.csv")

# df.head(n) df.tail(n) 查看前n条数据或者后n条数据

df.info() # 每列类型 + 有没有缺失
df.describe() # 统计
```

> 拿到数据**别急着洗**，先看这三眼，你才知道该洗哪。

### 2. 缺失值：该有的值没有，怎么办

```
df.isnull().sum() # 查每列缺少多少数据
df.dropna() # 把有缺失的行整个删除
df.fillna(0) # 给缺失数据填充0
df["col"].fillna(df["col"].mean()) # 用特定列均值填写特定列的缺失数据
```

### 3. 去重：同一条数据出现好几次

```
df.drop_duplicates() # 删除整行完全一样的, 仅仅保留第一次出现的一行
df.drop_duplicates(subset=["col"]) # 按某列去重
```

### 4. 类型转换：数字存成了文本

```
df["col"] = df["col"].astype(int)
df["col"] = pd.to_datetime(df["date"]) # 文本转日期
```

### 5. 异常值：一眼看穿离谱的数据 —— 筛选

```
df[(df["age"] > 100) & (df["city"] == "北京")] # 布尔抽取
```

### 6. 排序、分组、合并（日常高频三连）

```
df.sort_values("salary", ascending=False) # 按照工资从高到低排
df.groupby("city")["salary"].mean() # 每个城市平均工资

# 合并
pd.concat([df1, df2]) # 上下拼接
pd.merge(df1, df2, on="id", how="left") # 按id左连接 / right / inner / outer
```

> 这六个操作（缺失、去重、类型、异常、筛选、分组合并）就是数据清洗的「全套武功」。会了它们，市面上 80% 的清洗活你都能接。

### 7.补充

```
len(df) # 查看总条数
job_count = df["col"].value_counts() # 统计对应字段出现次数

def apply_func(col):
    if pd.isna(col):
        return "未知"
    return col.split(',')[0]
df["col"].apply(apply_func)
```

---

## 四、NumPy：藏在底下的算力引擎

Pandas 很多操作底层就是 NumPy 在跑。你只需要会这几招：

```
import numpy as np

arr = np.array([1, 2, 3])

arr.mean() # 均值
arr.max()
arr.min()
arr.sum()

arr = np.arange(10) # 快速生成 0-9

arr2 = arr * 2 # 每个元素操作 宏操作
```

> NumPy 的核心优势是**向量化**——不用 for 循环，一次性对整组数据计算，速度比纯 Python 快几十倍。你只要记住「能整组算，就别写循环」。

---

## 五、Matplotlib：看清数据长啥样

清洗前后，画个图你就知道「洗没洗干净」：

```
import matplotlib.pyplot as plt

# 折线图
plt.plot(df["col"], df["col"])
plt.show()

# 柱状图
plt.bar(df["col"], df["col"])
plt.show()

# 直方图
plt.hist(df["col"], bins=20) # bins 表示均分段数
plt.show()

# 散点图
plt.scatter(df["col"], df["col"])
plt.show()
```

> 记住一句：**别裸眼看表，画图看分布。** 一张直方图，比你盯 10 分钟表格更早发现异常值。

---

## 六、进阶提示（想接单、想变强再看）

1. **`apply` 函数**：对某一列做自定义处理，清洗里超常用

1. **`str` 方法**：`df['name'].str.strip()` 去空格、`str.contains()` 匹配，清洗文本必备

1. **性能**：数据量大时，能用 NumPy/内置方法就别写循环

1. **养成习惯**：清洗完 `df.info()` 复查一遍，确认类型和缺失都对了

---
