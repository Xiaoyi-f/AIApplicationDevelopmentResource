# 并查集与最小生成树：判断连通、连所有点的最小代价

> 我是小翊，19 岁，全干 AI 应用开发工程师。  
> 并查集（DSU）是图论里「最轻量却最常用」的工具，判断谁和谁连通、合并两个集合，几乎零学习成本却处处有用。最小生成树（MST）则直接建立在并查集之上。  
> 这一篇把并查集 + 最小生成树一次讲透，附完整可跑的 Python 实现。  
> Let's use AI to create a better world。

---

## 目录

- [一、并查集解决什么问题](#一并查集解决什么问题)
- [二、并查集两个核心操作](#二并查集两个核心操作)
- [三、两个关键优化](#三两个关键优化)
- [四、并查集完整实现](#四并查集完整实现)
- [五、最小生成树：Kruskal](#五最小生成树kruskal)
- [六、最小生成树：Prim](#六最小生成树prim)
- [七、速查表](#七速查表)

---

## 一、并查集解决什么问题

并查集专门处理**「元素属于哪个集合」**这类问题，支持两个操作：

1. **合并（union）**：把两个元素所在的集合合并成一个。
2. **查询（find）**：判断两个元素是否在同一个集合。

典型场景：

- 判断图里两个节点是否**连通**；
- 判断有没有形成**环**（加边前查两端是否已连通）；
- 统计**连通块数量**；
- 朋友圈、岛屿合并类问题。

> 一句话：**并查集 = 判断「谁和谁是一伙的」+「把两伙合并成一伙」。**

---

## 二、并查集两个核心操作

并查集的本质是**用树表示集合**，每个集合有一个「根」代表它：

- **find(x)**：一路往上找 x 的根节点。根相同 → 同一集合。
- **union(x, y)**：把 x 的根和 y 的根合并（让一个根指向另一个根）。

```python
# 最朴素的并查集
parent = [i for i in range(n)]  # 初始每个人是自己的根

def find(x):
    while parent[x] != x:  # 一直往上找根
        x = parent[x]
    return x

def union(x, y):
    rx, ry = find(x), find(y)
    if rx != ry:
        parent[rx] = ry  # 让 rx 的根指向 ry
```

---

## 三、两个关键优化

朴素并查集在极端情况下树会退化成**一条链**，`find` 变成 O(n)。两个优化能让它接近 O(1)：

### 优化 1：路径压缩（Path Compression）

`find` 时把沿途所有节点**直接指向根**，下次查就快了：

```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])  # 递归, 同时把路径压平
    return parent[x]
```

### 优化 2：按秩合并（Union by Rank）

合并时**矮树挂到高树上**，避免树越长越高：

```python
rank = [0] * n  # 记录每棵树的高度

def union(x, y):
    rx, ry = find(x), find(y)
    if rx == ry:
        return
    if rank[rx] < rank[ry]:
        parent[rx] = ry
    elif rank[rx] > rank[ry]:
        parent[ry] = rx
    else:
        parent[ry] = rx
        rank[rx] += 1
```

> 一句话：**路径压缩让「查」快，按秩合并让「树」矮。两个一起用，并查集近乎 O(1)。**

---

## 四、并查集完整实现

竞赛标准模板（路径压缩 + 按秩合并），背下来：

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.size = [1] * n  # 可选: 每个集合的大小

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False  # 已经在同一集合
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        self.size[rx] += self.size[ry]
        return True

    def connected(self, x, y):
        return self.find(x) == self.find(y)

if __name__ == "__main__":
    dsu = DSU(6)  # 6 个元素 0~5
    dsu.union(0, 1)
    dsu.union(1, 2)
    print(dsu.connected(0, 2))  # True
    print(dsu.connected(0, 3))  # False
    print(dsu.find(2))          # 根节点编号
```

> 关键点：**`union` 返回 `False` 表示两边原本就连通**，这个返回值在「判环」「Kruskal」里特别有用。

---

## 五、最小生成树：Kruskal

最小生成树（MST）求**用最小总代价连接所有节点**（n 个节点恰好 n-1 条边）。Kruskal 的思路极简单——**贪心 + 并查集**：

1. 把所有边按**权值从小到大**排序；
2. 依次取边，如果这条边**两端还没连通**（不在同一集合），就选它，并合并两端；
3. 选够 n-1 条边，就得到最小生成树。

```python
def kruskal(n, edges):
    # n 个节点编号 1~n, edges = [(u, v, w), ...]
    edges.sort(key=lambda e: e[2])  # 按权值升序
    dsu = DSU(n + 1)
    total = 0
    chosen = []
    for u, v, w in edges:
        if dsu.union(u, v):  # 两端不连通才选
            total += w
            chosen.append((u, v, w))
            if len(chosen) == n - 1:  # 已经连满
                break
    return total, chosen

if __name__ == "__main__":
    n = 5
    edges = [
        (1, 2, 3), (1, 3, 1), (2, 4, 2),
        (3, 4, 5), (3, 5, 1), (4, 5, 1)
    ]
    total, mst = kruskal(n, edges)
    print(total)  # 最小生成树总代价
    print(mst)    # 选出的边
```

**为什么贪心正确**：按权值从小到大选，只要不形成环（两端未连通）就选，最终一定是全局最优。这是 MST 的经典结论（贪心选择性质）。

> 一句话：**Kruskal = 边排序 + 并查集判环。每次选最小且不成环的边，选够 n-1 条。**

---

## 六、最小生成树：Prim

Prim 是另一种 MST 思路，从**一个起点出发不断扩展**，像「长蘑菇」一样把最近的节点接进来，类似 Dijkstra：

```python
import heapq

def prim(n, gw):
    # gw 带权邻接表, 节点 1~n
    INF = float("inf")
    visited = [False] * (n + 1)
    total = 0
    cnt = 0
    pq = [(0, 1)]  # (边权, 节点), 从节点 1 开始
    while pq and cnt < n:
        w, u = heapq.heappop(pq)
        if visited[u]:
            continue
        visited[u] = True
        total += w
        cnt += 1
        for v, ww in gw[u]:
            if not visited[v]:
                heapq.heappush(pq, (ww, v))
    return total
```

| 算法 | 思路 | 复杂度 | 适用 |
| --- | --- | --- | --- |
| **Kruskal** | 边排序 + 并查集 | O(m log m) | 边少（稀疏图） |
| **Prim** | 堆 + 逐步扩展 | O(m log n) | 边多（稠密图） |

> 一句话：**竞赛首选 Kruskal（简单、边稀疏时快），稠密图用 Prim。**

---

## 七、速查表

| 点 | 要点 |
| --- | --- |
| 并查集作用 | 判断连通、合并集合、统计连通块 |
| 两个操作 | find 查根、union 合并 |
| 两个优化 | 路径压缩、按秩合并 |
| 复杂度 | 近乎 O(1) |
| Kruskal | 边排序 + 并查集判环 |
| Prim | 堆 + 逐步扩展 |
| MST 本质 | n-1 条边连所有点的最小代价 |

---

**一句话总结**：并查集用「路径压缩 + 按秩合并」做到近乎 O(1) 的连通判断，是图论的瑞士军刀；最小生成树首选 Kruskal（边排序 + 并查集），求「连所有点的最小代价」。

**文末预告**：下一篇「拓扑排序与有向图判环」——处理任务依赖、课程表这类有向图问题。

Let's use AI to create a better world。
