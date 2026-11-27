# LRU 缓存：淘汰最久未使用的数据

> 我是小翊，19 岁，全干 AI 应用开发工程师。  
> LRU（Least Recently Used）是最经典、最高频的面试手写题之一。核心就一句：**淘汰很久没用过的数据，保留热点数据。**  
> 这一篇讲清 LRU 的原理、两种实现（OrderedDict 一行流 + 面试手写双向链表），代码都能直接跑。  
> Let's use AI to create a better world。

---

## 目录

- [一、LRU 是什么](#一lru-是什么)
- [二、核心规则](#二核心规则)
- [三、OrderedDict 实现（最快）](#三ordereddict-实现最快)
- [四、手写双向链表 + 哈希表（面试版）](#四手写双向链表--哈希表面试版)
- [五、LRU 速查表](#五lru-速查表)

---

## 一、LRU 是什么

**LRU = Least Recently Used，最近最少使用。**

是一种**缓存淘汰策略**：当缓存满了，要腾出空间时，**优先淘汰「最久没用过」的数据**，保留「最近刚用过」的热点数据。

> 一句话：**谁最久没被访问，就先淘汰谁。** 热数据留下，冷数据走人。

---

## 二、核心规则

LRU 只有两个操作，规则很简单：

| 操作 | 规则 |
| --- | --- |
| **get(key)** | 查到 → 返回，并把它标记为「最近使用」（移到队尾）；查不到 → 返回 None |
| **put(key, value)** | 存入；若 key 已存在则更新并标记最近使用；若容量满了则淘汰「最久未用」（队头） |

**关键认知**：每次 `get` 或 `put` 命中时，都要把该元素**移到「最近使用」的位置**——这是 LRU 的灵魂。

---

## 三、OrderedDict 实现（最快）

Python 的 `OrderedDict` 能记住插入顺序，且 `move_to_end` 能把元素移到末尾，正好实现 LRU。

```python
# LRU缓存
from collections import OrderedDict

class LRUCache:

    def __init__(self, capacity: int):
        self.cap = capacity
        self.cache = OrderedDict()

    def get(self, key):
        if key not in self.cache:
            return None

        self.cache.move_to_end(key)

        return self.cache[key]

    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)

        self.cache[key] = value

        if len(self.cache) > self.cap:
            self.cache.popitem(last=False)

# 测试
if __name__ == "__main__":
    lru = LRUCache(3)
    lru.put("A", 1)
    lru.put("B", 2)
    lru.put("C", 3)
    print(lru.get("A"))  # 1，且 A 被移到最近使用
    lru.put("D", 4)      # 满了，淘汰最久未用的 B
    print(lru.cache)     # OrderedDict([('C', 3), ('A', 1), ('D', 4)])
```

**核心理解（易错）**：
- `move_to_end(key)`：把 key 移到「最近使用」的末尾。
- `popitem(last=False)`：弹出「最久未用」的队头（`last=False` 表示弹第一个）。
- 队尾 = 最近使用，队头 = 最久未用。

---

## 四、手写双向链表 + 哈希表（面试版）

面试官通常**不让用 OrderedDict**，要求手写。标准解法是「**双向链表 + 哈希表**」：

- **哈希表**：key → 链表节点，O(1) 查找。
- **双向链表**：维护使用顺序，O(1) 移动/删除节点。

```python
class DLinkedNode:
    def __init__(self, key=0, value=0):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.cache = {}  # key -> DLinkedNode
        # 哑头、哑尾，简化边界处理
        self.head = DLinkedNode()
        self.tail = DLinkedNode()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _add_to_tail(self, node):
        # 插到队尾（最近使用）
        node.prev = self.tail.prev
        node.next = self.tail
        self.tail.prev.next = node
        self.tail.prev = node

    def get(self, key):
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._remove(node)
        self._add_to_tail(node)
        return node.value

    def put(self, key, value):
        if key in self.cache:
            node = self.cache[key]
            node.value = value
            self._remove(node)
            self._add_to_tail(node)
        else:
            node = DLinkedNode(key, value)
            self.cache[key] = node
            self._add_to_tail(node)
            if len(self.cache) > self.cap:
                # 淘汰队头（最久未用）
                oldest = self.head.next
                self._remove(oldest)
                del self.cache[oldest.key]
```

**核心理解（面试必背）**：
- **双向链表**让「移动节点」「删除节点」都 O(1)，**哈希表**让「按 key 找到节点」O(1)。
- **哑头 + 哑尾**（dummy head/tail）简化边界处理，不用判断空链表。
- 队头 = 最久未用，队尾 = 最近使用。

---

## 五、LRU 速查表

| 点 | 要点 |
| --- | --- |
| 本质 | 淘汰最久未使用的数据 |
| 两个操作 | get（命中移到最近使用）、put（满了淘汰队头） |
| 最快实现 | OrderedDict + move_to_end + popitem(last=False) |
| 面试手写 | 双向链表 + 哈希表 |
| 复杂度 | get/put 都是 O(1) |
| 哑头哑尾 | 简化边界处理 |

---

**一句话总结**：LRU 就是「**谁最久没用就淘汰谁**」。OrderedDict 一行流解决日常，手写「双向链表 + 哈希表」应付面试，核心都是「每次访问把元素移到队尾」。

**文末预告**：下一篇「有限状态机」——用跳转表把复杂的 if-else 状态流转梳理成清晰的结构。

Let's use AI to create a better world。
