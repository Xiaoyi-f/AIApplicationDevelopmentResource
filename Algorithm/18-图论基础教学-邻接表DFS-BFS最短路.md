# 图论基础：邻接表建图 + DFS/BFS + 最短路一篇讲透

> 我是小翊，19 岁，全干 AI 应用开发工程师。  
> 图论是百度之星**必考且占比最大**的模块，几乎每场都有图题。最短路、连通性、拓扑排序、最小生成树，全都建立在「会建图、会遍历」之上。  
> 这一篇打地基：邻接表建图、DFS/BFS、Dijkstra 和 Floyd 两种最短路，附完整可跑的 Python 实现（含性能优化）。  
> Let's use AI to create a better world。

---

## 目录

- [一、图怎么存：邻接矩阵 vs 邻接表](#一图怎么存邻接矩阵-vs-邻接表)
- [二、DFS 深度优先搜索](#二dfs深度优先搜索)
- [三、BFS 广度优先搜索](#三bfs广度优先搜索)
- [四、Dijkstra 单源最短路](#四dijkstra单源最短路)
- [五、Floyd 全源最短路](#五floyd全源最短路)
- [六、DFS/BFS 对比速查](#六dfsbfs对比速查)
- [七、最短路算法选择速查](#七最短路算法选择速查)

---

## 一、图怎么存：邻接矩阵 vs 邻接表

图由**顶点（节点）**和**边（连接）**组成。存图有两种主流方式：

| 方式 | 结构 | 空间 | 适用 |
| --- | --- | --- | --- |
| **邻接矩阵** | 二维数组 `g[i][j]` | O(n²) | 边密集、要快速判断两点相连 |
| **邻接表** | `list[list]` 存邻居 | O(n+m) | 边稀疏、竞赛最常用 |

竞赛图通常**边稀疏**，一律用邻接表。Python 里邻接表就是一个「列表套列表」：

```python
# 邻接表建图 (无向图)
n = 5  # 5 个节点, 编号 1~5
edges = [(1, 2), (1, 3), (2, 4), (3, 5)]

g = [[] for _ in range(n + 1)]  # 节点从 1 开始, 多开一位
for u, v in edges:
    g[u].append(v)
    g[v].append(u)  # 无向图加双向边

# 带权图: 每个邻居存 (邻居节点, 边权)
gw = [[] for _ in range(n + 1)]
for u, v, w in [(1, 2, 3), (1, 3, 1), (2, 4, 2)]:
    gw[u].append((v, w))
    gw[v].append((u, w))  # 无向图
```

> 一句话：**竞赛图用邻接表（列表套列表），带权图存 `(邻居, 权值)` 元组。** 节点编号不连续时，用 dict 建邻接表。

---

## 二、DFS 深度优先搜索

DFS 的核心是**一路走到底，再回头**，用**递归**（或显式栈）实现。适合「判断连通性」「求路径」「遍历整张图」。

```python
import sys
sys.setrecursionlimit(10 ** 6)

def dfs(u, g, visited):
    visited[u] = True
    print(u, end=" ")  # 访问节点
    for v in g[u]:
        if not visited[v]:
            dfs(v, g, visited)

# 遍历整张图 (可能不连通)
def dfs_all(g, n):
    visited = [False] * (n + 1)
    for u in range(1, n + 1):
        if not visited[u]:
            dfs(u, g, visited)
```

**两种经典应用**：

```python
# 应用1: 判断两个节点是否连通
def is_connected(g, s, t, n):
    visited = [False] * (n + 1)
    stack = [s]
    visited[s] = True
    while stack:
        u = stack.pop()
        for v in g[u]:
            if not visited[v]:
                visited[v] = True
                stack.append(v)
    return visited[t]

# 应用2: 求连通块数量
def count_components(g, n):
    visited = [False] * (n + 1)
    cnt = 0
    for u in range(1, n + 1):
        if not visited[u]:
            cnt += 1
            stack = [u]
            visited[u] = True
            while stack:
                x = stack.pop()
                for v in g[x]:
                    if not visited[v]:
                        visited[v] = True
                        stack.append(v)
    return cnt
```

> 一句话：**DFS 一路走到底，用递归（或栈）实现，适合连通性、连通块、找路径。**

---

## 三、BFS 广度优先搜索

BFS 的核心是**一层一层往外扩**，用**队列**实现。它有一个 DFS 没有的关键性质：**第一次到达某节点时走的步数，就是最短步数**（边权都为 1 时）。

```python
from collections import deque

def bfs(g, s, n):
    # 求 s 到其他所有节点的最短步数 (边权都为 1)
    dist = [-1] * (n + 1)
    dist[s] = 0
    q = deque([s])
    while q:
        u = q.popleft()
        for v in g[u]:
            if dist[v] == -1:  # 没访问过
                dist[v] = dist[u] + 1
                q.append(v)
    return dist
```

**为什么 BFS 能求最短步数**：因为 BFS 按「距离 0、1、2、3...」的顺序逐层访问，第一次走到某个节点，走的就是最短路径。

> 一句话：**BFS 一层层往外扩，用队列实现。边权为 1 时，BFS 天然能求最短步数，这是它最重要的性质。**

---

## 四、Dijkstra 单源最短路

Dijkstra 求**一个起点到所有其他点的最短距离**，要求**边权非负**。核心是「贪心 + 优先队列」：每次取出当前距离最小的未确定节点，用它去松弛邻居。

```python
import heapq

def dijkstra(gw, s, n):
    # gw 是带权邻接表, gw[u] = [(v, w), ...]
    INF = float("inf")
    dist = [INF] * (n + 1)
    dist[s] = 0
    pq = [(0, s)]  # (距离, 节点), 小顶堆
    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]:  # 这是过期的旧记录, 跳过
            continue
        for v, w in gw[u]:
            nd = d + w
            if nd < dist[v]:  # 松弛: 找到更短的路
                dist[v] = nd
                heapq.heappush(pq, (nd, v))
    return dist

if __name__ == "__main__":
    n = 5
    gw = [[] for _ in range(n + 1)]
    edges = [(1, 2, 3), (1, 3, 1), (2, 4, 2), (3, 4, 5), (3, 5, 1), (4, 5, 1)]
    for u, v, w in edges:
        gw[u].append((v, w))
        gw[v].append((u, w))
    print(dijkstra(gw, 1, n))  # [inf, 0, 3, 1, 5, 2]
```

**核心理解**：

| 要点 | 说明 |
| --- | --- |
| `dist[u]` | 起点到 u 的当前最短距离 |
| **松弛** | 用 u 去更新邻居 v：`dist[v] = min(dist[v], dist[u] + w)` |
| **小顶堆** | 每次取距离最小的节点，堆里存 `(距离, 节点)` |
| `if d > dist[u]` | 跳过「已经找到更短路径」的过期记录，防止重复处理 |
| 复杂度 | O((n+m) log n) |

> 一句话：**Dijkstra = 贪心 + 堆。每次取最近的点去松弛邻居，边权非负时正确。**

**用 Python 打竞赛时**：堆里存元组 `(距离, 节点)` 会自动按第一个元素（距离）排序，天然是小顶堆。这个写法是竞赛标准模板，背下来。

---

## 五、Floyd 全源最短路

Floyd 求**任意两点之间**的最短距离，用**三重循环动态规划**。代码极短，但复杂度 O(n³)，只适合**节点数少（n ≤ 几百）**的图。

```python
def floyd(n, edges):
    # n 个节点编号 1~n, edges = [(u, v, w), ...]
    INF = float("inf")
    dist = [[INF] * (n + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        dist[i][i] = 0  # 自己到自己距离 0
    for u, v, w in edges:
        dist[u][v] = min(dist[u][v], w)  # 处理重边, 取最小

    # 三重循环: 以 k 为中转点
    for k in range(1, n + 1):
        for i in range(1, n + 1):
            for j in range(1, n + 1):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    return dist
```

**核心思想**：`dist[i][j]` 表示「只经过编号 ≤ k 的中转点时，i 到 j 的最短距离」。三重循环把每个点都当成「中转站」尝试一遍。

> 一句话：**Floyd 代码短、能求全源最短路，但 O(n³)，只适合 n ≤ 几百 的小图。**

---

## 六、DFS/BFS 对比速查

| 维度 | DFS | BFS |
| --- | --- | --- |
| 实现 | 递归 / 栈 | 队列 |
| 顺序 | 一路到底 | 一层一层 |
| 求最短步数 | ❌ 不能保证 | ✅（边权为 1） |
| 连通性/连通块 | ✅ | ✅ |
| 找路径/回溯 | ✅ 更顺手 | 需要记父节点 |
| 空间 | 栈深度 | 队列宽度 |

---

## 七、最短路算法选择速查

| 算法 | 单源/全源 | 复杂度 | 适用 |
| --- | --- | --- | --- |
| **BFS** | 单源 | O(n+m) | 边权全为 1 |
| **Dijkstra** | 单源 | O((n+m)log n) | 边权非负 |
| **Floyd** | 全源 | O(n³) | n 小、求任意两点 |
| **SPFA** | 单源 | 平均快、可能退化 | 有负权边 |

> 竞赛直觉：**先看边权是不是 1**——是就 BFS；**边权非负**——Dijkstra；**有负权**——SPFA；**n 很小要求全源**——Floyd。

---

**一句话总结**：图论地基就三件事——**邻接表建图、DFS/BFS 遍历、最短路（Dijkstra/Floyd）**。先把这三样练到「闭眼能写」，后面拓扑排序、最小生成树都是它们的组合拳。

**文末预告**：下一篇「并查集与最小生成树」——路径压缩、按秩合并，Kruskal/Prim 连所有点的最小代价。

Let's use AI to create a better world。
