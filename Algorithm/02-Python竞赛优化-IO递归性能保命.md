# Python 打竞赛：I/O、递归、性能保命技巧一篇讲透

> 我是小翊，19 岁，全干 AI 应用开发工程师。  
> 用 Python 打百度之星，最大的敌人不是算法，是**超时（TLE）**。同样的思路，C++ 能过，Python 可能卡在 2 秒的时限上。  
> 这一篇不讲算法，专讲 Python 竞赛的「性能保命技巧」：快速 I/O、递归深度、PyPy、内置函数、避坑清单。掌握它，你的 Python 才能稳稳过题。  
> Let's use AI to create a better world。

---

## 目录

- [一、为什么 Python 容易超时](#一为什么python容易超时)
- [二、快速输入输出（最救命）](#二快速输入输出最救命)
- [三、递归深度](#三递归深度)
- [四、能用内置就别自己写](#四能用内置就别自己写)
- [五、常见超时陷阱清单](#五常见超时陷阱清单)
- [六、PyPy 与评测环境](#六pypy与评测环境)
- [七、一个完整竞赛模板](#七一个完整竞赛模板)
- [八、速查表](#八速查表)

---

## 一、为什么 Python 容易超时

Python 是**解释型语言**，天生比 C++ 慢 10~100 倍。同样 10 万次循环，C++ 毫秒级，Python 可能上百毫秒。竞赛题限时通常 1~2 秒，数据量一大，Python 就会 TLE。

**核心矛盾**：Python 写起来快，跑起来慢。

**应对思路**：
1. 把**最慢的部分**（I/O、重复计算）优化掉；
2. 能用**内置函数/C 实现**的，绝不自己写 Python 循环；
3. 实在过不了，就承认「这题 Python 不友好」，用更巧的算法降复杂度。

> 一句话：**Python 打竞赛 = 算法功底 + 常数优化。算法决定上限，常数决定能不能过。**

---

## 二、快速输入输出（最救命）

默认的 `input()` 和 `print()` 很慢，数据一大就拖后腿。竞赛必须用 `sys.stdin.buffer` 读，`sys.stdout.write` 写。

```python
import sys

def main():
    # 一次性读入全部数据, 按空白切分
    data = sys.stdin.buffer.read().split()
    # 逐个转成 int
    n = int(data[0])
    m = int(data[1])
    # ... 处理逻辑

    # 输出: 拼成一个字符串一次性写
    out = []
    for i in range(1, n + 1):
        out.append(str(i * i))
    sys.stdout.write("\n".join(out) + "\n")

if __name__ == "__main__":
    main()
```

**核心要点**：

| 写法 | 快慢 | 说明 |
| --- | --- | --- |
| `input()` | 慢 | 每次调用有开销 |
| `sys.stdin.readline()` | 中 | 逐行读，一般够用 |
| `sys.stdin.buffer.read().split()` | **最快** | 一次性全读入切分 |
| `print()` 每行 | 慢 | 频繁调用有开销 |
| `sys.stdout.write("\n".join(list))` | **最快** | 拼好一次性写 |

> 铁律：**输入用 `sys.stdin.buffer.read().split()`，输出用 `"\n".join` 一次性写。** 这是 Python 过题的第一道保险。

**逐行读的折中写法**（数据太大不能一次性读时）：

```python
import sys
input = sys.stdin.readline  # 用 readline 替代 input

n = int(input())
for _ in range(n):
    a, b = map(int, input().split())
```

---

## 三、递归深度

Python 默认递归深度约 1000，树的 DFS、深搜很容易爆 `RecursionError`。竞赛一定要设大：

```python
import sys
sys.setrecursionlimit(10 ** 6)  # 设到 100 万
```

**注意**：设大了只是「允许」深递归，但递归太深仍可能爆栈。遇到递归会超深的题（如链式 DFS），最好改成**显式栈**（用 list 模拟）。

> 一句话：**第一行先写 `sys.setrecursionlimit(10**6)`，再写 `sys.setrecursionlimit` 也救不了的深度，就改迭代。**

---

## 四、能用内置就别自己写

Python 的内置函数和标准库是 C 写的，比你手写 Python 循环快几十倍。**这是 Python 竞赛的灵魂。**

| 场景 | 慢写法 | 快写法 |
| --- | --- | --- |
| 求和 | `sum=0; for x in a: sum+=x` | `sum(a)` |
| 求最大最小 | `for` 循环比 | `max(a)` / `min(a)` |
| 排序 | 手写快排 | `a.sort()` |
| 反转 | 手写 | `a.reverse()` 或 `a[::-1]` |
| 去重排序 | 手写 | `sorted(set(a))` |
| 计数 | 手写 dict 累加 | `collections.Counter(a)` |
| 双端队列 | 用 list 模拟 | `collections.deque` |
| 堆 | 手写 | `heapq` |
| 排列组合 | 手写 | `itertools.permutations/combinations` |
| 二分 | 手写 | `bisect` |

```python
import heapq
import bisect
from collections import Counter, deque

# 堆: 找第 k 小
heap = [5, 1, 8, 3]
heapq.heapify(heap)          # 原地建堆 O(n)
smallest = heapq.heappop(heap)  # 弹出最小

# 二分: 在有序列表里找插入位置
arr = [1, 3, 5, 7, 9]
pos = bisect.bisect_left(arr, 5)  # 第一个 >=5 的位置

# 计数: 统计每个元素出现次数
cnt = Counter([1, 2, 2, 3, 3, 3])
print(cnt[3])  # 3

# 双端队列: 两端都能 O(1) 操作
dq = deque([1, 2, 3])
dq.appendleft(0)   # 左边加
dq.pop()           # 右边弹
```

> 铁律：**能想到「Python 有没有现成的函数干这个」，比「我怎么写这个」更重要。**

---

## 五、常见超时陷阱清单

这些是 Python 竞赛最容易踩的坑，逐条对照：

| 陷阱 | 问题 | 解法 |
| --- | --- | --- |
| `str` 拼接循环 | `s += x` 每次新建字符串 O(n²) | 用 `list` 收集，最后 `"".join` |
| 字符串 `in` 判断子串 | 频繁 `s in t` 慢 | 用 `set` 存集合，`in` 是 O(1) |
| list 头插 | `a.insert(0, x)` 是 O(n) | 用 `deque` 的 `appendleft` |
| 大整数滥用 | Python 大整数运算有额外开销 | 能取模就取模，别让数无限膨胀 |
| 二维 list 手写循环 | 双重 for 慢 | 尽量用内置、列表推导式 |
| 重复求值 | 循环里重复算同一表达式 | 提到循环外缓存 |
| 用 `float` 比大小 | 浮点精度/速度问题 | 能整数就整数 |

**列表推导式通常比 for 循环快**：

```python
# 慢
squares = []
for i in range(1000000):
    squares.append(i * i)

# 快
squares = [i * i for i in range(1000000)]
```

> 一句话：**先扫一遍陷阱清单，再优化，往往比优化算法本身更早见效。**

---

## 六、PyPy 与评测环境

- **PyPy** 是 Python 的 JIT（即时编译）实现，跑纯计算类循环比 CPython 快数倍。竞赛平台通常支持 PyPy，**能用 PyPy 就选 PyPy 提交**。
- 注意：PyPy 对「调用 C 扩展」（如 numpy）反而可能慢，但竞赛一般用不到 numpy，纯算法题 PyPy 是更优解。
- **Python 版本**：主流平台支持 Python 3。`f-string`、`sort` 的 `key=` 都可用。

> 一句话：**纯算法题优先选 PyPy，能用 PyPy 不用 CPython。**

---

## 七、一个完整竞赛模板

下面这个模板可以直接当「每次开题的骨架」用：

```python
import sys
sys.setrecursionlimit(10 ** 6)

def solve():
    # 快速读入
    data = sys.stdin.buffer.read().split()
    it = iter(data)
    n = int(next(it))

    # ==== 你的算法逻辑 ====
    result = []

    # ==== 快速输出 ====
    sys.stdout.write("\n".join(result) + "\n")

if __name__ == "__main__":
    solve()
```

> 用 `iter(data)` + `next(it)` 可以顺序取数，比手动维护下标更清晰。

---

## 八、速查表

| 点 | 要点 |
| --- | --- |
| 输入 | `sys.stdin.buffer.read().split()` |
| 输出 | `"\n".join(list)` 一次性写 |
| 递归 | `sys.setrecursionlimit(10**6)` |
| 排序 | `a.sort()`，用 `key=` 定制 |
| 堆 | `heapq` |
| 二分 | `bisect` |
| 计数 | `collections.Counter` |
| 双端队列 | `collections.deque` |
| 拼接 | 先 `list` 再 `"".join` |
| 提交 | 能用 PyPy 就选 PyPy |

---

**一句话总结**：用 Python 打百度之星，算法是内功，性能优化是保命符。记住四条——**快读快写、设递归深度、内置函数优先、避开拼接/头插陷阱**，再配合 PyPy，Python 也能稳稳过初赛、冲决赛。

**文末预告**：下一篇「图论基础」——邻接表建图 + DFS/BFS + Dijkstra/Floyd 最短路，百度之星必考，一次讲透。

Let's use AI to create a better world。
