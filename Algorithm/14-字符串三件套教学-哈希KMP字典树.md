# 字符串三件套：哈希 + KMP + 字典树，匹配与检索一次讲透

> 我是小翊，19 岁，全干 AI 应用开发工程师。  
> 字符串题在百度之星里高频出现，官方考点明确列了「KMP、AC 自动机、后缀数组」。这一篇先吃透三个最核心的：**字符串哈希、KMP、字典树（Trie）**。  
> 它们覆盖了字符串的三大诉求——**快速比较、快速匹配、快速前缀检索**。附完整可跑的 Python 实现。  
> Let's use AI to create a better world。

---

## 目录

- [一、字符串哈希：O(1) 比较子串](#一字符串哈希o1比较子串)
- [二、KMP：线性时间模式匹配](#二kmp线性时间模式匹配)
- [三、字典树 Trie：前缀检索](#三字典树trie前缀检索)
- [四、三件套对比](#四三件套对比)
- [五、速查表](#五速查表)

---

## 一、字符串哈希：O(1) 比较子串

字符串哈希的核心思想：**把字符串看成一个「进制数」，算出一个哈希值**。比较两个子串是否相等，就变成比较两个哈希值，O(1) 完成。

```python
def build_hash(s, base=131, mod=10 ** 9 + 7):
    # h[i] = s[0..i-1] 的哈希值, p[i] = base^i
    n = len(s)
    h = [0] * (n + 1)
    p = [1] * (n + 1)
    for i in range(n):
        h[i + 1] = (h[i] * base + ord(s[i])) % mod
        p[i + 1] = (p[i] * base) % mod
    return h, p

def get_hash(h, p, l, r, mod=10 ** 9 + 7):
    # 子串 s[l..r] (从 0 开始) 的哈希值
    return (h[r + 1] - h[l] * p[r - l + 1]) % mod

if __name__ == "__main__":
    s = "ababa"
    h, p = build_hash(s)
    # 判断 s[0..1]="ab" 和 s[2..3]="ab" 是否相等
    print(get_hash(h, p, 0, 1) == get_hash(h, p, 2, 3))  # True
```

**核心公式**（背下来）：

```
子串 [l, r] 哈希 = h[r+1] - h[l] * base^(r-l+1)
```

**为什么快**：提前算好每个前缀的哈希，任意子串哈希用「减法 + 乘一个幂」一步得出。哈希相等就（大概率）字符串相等。

**防冲突技巧**：用**双哈希**（两个不同的 base/mod 各算一遍），都相等才判定相等，冲突概率几乎为 0。

> 一句话：**字符串哈希把「比较子串」从 O(n) 降到 O(1)，是回文、最长重复子串、字符串判重题的利器。**

---

## 二、KMP：线性时间模式匹配

KMP 解决**「在一个长文本里找模式串」**的问题。朴素做法是暴力逐位比较，最坏 O(nm)；KMP 利用**已匹配的信息**，做到 O(n+m)。

**核心是 next 数组**（也叫前缀函数）：`next[i]` 表示「模式串前 i 个字符里，最长的「相同前缀和后缀」的长度」。匹配失败时，不用从头再来，而是跳到 next 记录的位置。

```python
def build_next(p):
    # 求 next 数组, next[i] = 前 i 个字符的最长公共前后缀长度
    m = len(p)
    nxt = [0] * m
    j = 0
    for i in range(1, m):
        while j > 0 and p[i] != p[j]:
            j = nxt[j - 1]
        if p[i] == p[j]:
            j += 1
        nxt[i] = j
    return nxt

def kmp(s, p):
    # 返回模式串 p 在文本 s 中出现的所有起始位置
    nxt = build_next(p)
    res = []
    j = 0
    for i in range(len(s)):
        while j > 0 and s[i] != p[j]:
            j = nxt[j - 1]  # 关键: 失配后回退, 不回退 i
        if s[i] == p[j]:
            j += 1
        if j == len(p):     # 匹配成功
            res.append(i - len(p) + 1)
            j = nxt[j - 1]  # 继续找下一个
    return res

if __name__ == "__main__":
    s = "abababac"
    p = "aba"
    print(kmp(s, p))  # [0, 2, 4]
```

**KMP 的精髓**：失配时，`i`（文本指针）**只前进不后退**，靠 `j` 通过 next 数组回退。这样文本每个字符最多被比较常数次，整体 O(n+m)。

> 一句话：**KMP 靠 next 数组记录「最长公共前后缀」，失配时 j 回退、i 不回退，把匹配降到 O(n+m)。**

---

## 三、字典树 Trie：前缀检索

字典树（Trie）是**「用树存一堆字符串」**的结构，专门加速**前缀相关的查询**（自动补全、前缀匹配、找公共前缀）。

```python
class TrieNode:
    def __init__(self):
        self.children = {}          # 子节点: 字符 -> 节点
        self.is_end = False         # 是否是某个单词的结尾

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word):
        # 判断 word 是否完整存在
        node = self.root
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return node.is_end

    def starts_with(self, prefix):
        # 判断是否有以 prefix 开头的单词
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return True

if __name__ == "__main__":
    t = Trie()
    for w in ["app", "apple", "banana"]:
        t.insert(w)
    print(t.search("app"))        # True
    print(t.search("appl"))       # False
    print(t.starts_with("app"))   # True
```

**复杂度**：插入和查询都是 O(L)，L 是单词长度，与「存了多少单词」无关。这是它比「暴力遍历所有字符串」快的关键。

> 一句话：**Trie 把字符串按「字符逐个分叉」存成树，前缀检索 O(L)，是自动补全、前缀匹配题的标配。**

---

## 四、三件套对比

| 技巧 | 解决 | 复杂度 | 典型场景 |
| --- | --- | --- | --- |
| **字符串哈希** | 子串是否相等 | 查询 O(1) | 回文、最长重复子串、判重 |
| **KMP** | 模式串匹配 | O(n+m) | 找子串出现位置 |
| **Trie** | 前缀检索 | O(L) | 自动补全、前缀匹配、公共前缀 |

> 竞赛直觉：**「两个子串比相等」→ 哈希；「找一个串在另一个串里的位置」→ KMP；「一堆串按前缀查」→ Trie。**

---

## 五、速查表

| 点 | 要点 |
| --- | --- |
| 字符串哈希 | 字符串当进制数，子串哈希 O(1) |
| 哈希公式 | `h[r+1] - h[l] * base^(r-l+1)` |
| 防冲突 | 双哈希 |
| KMP | next 数组 + i 不回退 |
| KMP 复杂度 | O(n+m) |
| Trie | 字符逐个分叉，前缀检索 O(L) |
| 进阶 | AC 自动机 = KMP + Trie |

---

**一句话总结**：字符串三件套各司其职——**哈希管「快速比较」、KMP 管「快速匹配」、Trie 管「前缀检索」**。掌握这三个，字符串题的大半江山就稳了；再进阶就是它们的合体「AC 自动机」。

**文末预告**：下一篇「树状数组与线段树」——区间修改 + 区间查询的高级数据结构，动态维护区间的利器。

Let's use AI to create a better world。
