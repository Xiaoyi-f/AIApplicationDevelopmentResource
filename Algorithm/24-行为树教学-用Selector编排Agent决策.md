# 行为树：用 Sequence / Selector 编排 Agent 的决策逻辑

> 我是小翊，19 岁，全干 AI 应用开发工程师。  
> 行为树（Behavior Tree，BT）是组织 Agent 行为决策的一种结构，游戏 AI、机器人、大模型 Agent 都在用。  
> 这一篇讲清它的三种核心节点：顺序节点、选择节点、叶子节点，用一个「图谱问答 Agent」的真实场景串起来，附完整可跑的 Python 实现。  
> Let's use AI to create a better world。

---

## 目录

- [一、为什么需要行为树](#一为什么需要行为树)
- [二、三种核心节点](#二三种核心节点)
- [三、节点返回状态](#三节点返回状态)
- [四、Sequence 顺序节点](#四sequence顺序节点)
- [五、Selector 选择节点](#五selector选择节点)
- [六、叶子节点：对接真实能力](#六叶子节点对接真实能力)
- [七、完整实现：图谱问答 Agent](#七完整实现图谱问答agent)
- [八、行为树速查表](#八行为树速查表)

---

## 一、为什么需要行为树

一个大模型 Agent 不是「调用一次大模型」就完事了，它背后通常是一连串有顺序、有分支、有兜底的决策：

```
校验用户提问 → 检索知识图谱 → 校验置信度 → 生成回答
                                     ↘ 检索失败 → 直接泛回答兜底
```

如果纯用 `if-else` 硬写，逻辑会变成一坨「谁依赖谁、哪个失败走哪条」的乱麻，加一个分支要翻遍所有代码。

**行为树把「Agent 怎么做决策」抽象成一棵树**：

- 树的**内部节点**负责「怎么组织子节点」（顺序执行 / 选择执行）；
- 树的**叶子节点**负责「真正干活的动作」（调 Neo4j、调大模型）；
- 每个节点执行后返回一个**状态**（成功 / 失败 / 运行中），父节点根据子节点状态决定下一步。

> 一句话：**行为树 = 树形结构 + 三态返回，把 Agent 的决策逻辑变成可编排、可复用、可兜底的组合。**

---

## 二、三种核心节点

行为树最核心的节点就三种，理解这三种，90% 的行为树你都会搭：

| 节点 | 作用 | 返回逻辑 |
| --- | --- | --- |
| **Sequence（顺序节点）** | 子节点**依次**执行，全成功才成功 | 一个失败整体失败，一个运行中则暂停 |
| **Selector（选择节点）** | 子节点**挨个尝试**，第一个成功就成功 | 全部失败才失败 |
| **Leaf（叶子节点）** | 执行**真实动作**（调后端能力） | 返回成功 / 失败 / 运行中 |

> 记忆口诀：**Sequence 是「AND」（都要成），Selector 是「OR」（有一个成就行）。**

---

## 三、节点返回状态

每个节点执行完都要返回一个状态，父节点靠它来做决策：

```python
from enum import Enum

# 节点返回状态
class NodeStatus(Enum):
    SUCCESS = "SUCCESS"   # 成功
    FAIL = "FAIL"         # 失败
    RUNNING = "RUNNING"   # 运行中（异步任务还没做完）
```

**三态的含义**：

| 状态 | 含义 | 父节点怎么处理 |
| --- | --- | --- |
| `SUCCESS` | 这一步做成了 | 继续下一步 / 判定整体成功 |
| `FAIL` | 这一步失败了 | Sequence 整体失败 / Selector 试下一个 |
| `RUNNING` | 还没做完，下轮再问 | 父节点暂停，返回 RUNNING |

> `RUNNING` 是为**异步/多帧任务**设计的（比如游戏 AI 每帧 tick 一次）。同步场景里基本只用 SUCCESS / FAIL。

---

## 四、Sequence 顺序节点

Sequence 的语义：**子节点依次执行，一个失败整体失败，全部成功才成功。**

```python
class BTNode:
    """行为树基础节点"""
    def tick(self):
        raise NotImplementedError


class Sequence(BTNode):
    """顺序节点: 子节点依次执行, 一个失败整体失败, 全部成功才成功"""
    def __init__(self, children):
        self.children = children

    def tick(self):
        for child in self.children:
            res = child.tick()
            if res == NodeStatus.FAIL:
                return NodeStatus.FAIL
            if res == NodeStatus.RUNNING:
                return NodeStatus.RUNNING
        return NodeStatus.SUCCESS
```

**执行逻辑**：

1. 从左到右依次 `tick` 每个子节点；
2. 遇到 `FAIL` → 立刻返回 `FAIL`（短路，后面不执行）；
3. 遇到 `RUNNING` → 立刻返回 `RUNNING`（暂停）；
4. 全部走完都没失败 → 返回 `SUCCESS`。

> 典型用法：**「校验提问 → 检索图谱 → 校验置信度 → 生成回答」**，这是一串必须全部成功的流水线，用 Sequence 正合适。

---

## 五、Selector 选择节点

Selector 的语义：**挨个尝试子节点，遇到第一个成功就返回成功，全部失败才失败。**

```python
class Selector(BTNode):
    """选择节点: 挨个尝试, 遇到第一个成功就返回成功, 全部失败才失败"""
    def __init__(self, children):
        self.children = children

    def tick(self):
        for child in self.children:
            res = child.tick()
            if res == NodeStatus.SUCCESS:
                return NodeStatus.SUCCESS
            if res == NodeStatus.RUNNING:
                return NodeStatus.RUNNING
        return NodeStatus.FAIL
```

**执行逻辑**：

1. 从左到右依次尝试子节点；
2. 遇到 `SUCCESS` → 立刻返回 `SUCCESS`（短路，后面不试了）；
3. 遇到 `RUNNING` → 返回 `RUNNING`；
4. 全部失败 → 返回 `FAIL`。

> 典型用法：**「优先走图谱链路，图谱挂了就走大模型兜底」**。这就是 Selector 的「备选方案 / fallback」能力。

---

## 六、叶子节点：对接真实能力

叶子节点是树的「末端动作」，每个都对接你真实的后端能力。下面这几个就是真实场景里的叶子节点：

```python
class ValidateQuery(BTNode):
    """叶子: 校验用户提问是否合法, 过滤无效、违规问题"""
    def __init__(self, query):
        self.query = query

    def tick(self):
        print(f"[BT] 校验用户提问：{self.query}")
        if len(self.query.strip()) > 2:
            return NodeStatus.SUCCESS
        print("[BT] 问题无效，太短")
        return NodeStatus.FAIL


class GraphRetrieve(BTNode):
    """叶子: Neo4j知识图谱检索（GraphRAG）"""
    def __init__(self, query):
        self.query = query

    def tick(self):
        print(f"[BT] 调用Neo4j图谱检索：{self.query}")
        # 这里替换成你真实的cypher查询代码
        mock_retrieve_ok = True
        if mock_retrieve_ok:
            print("[BT] 图谱检索拿到关系链路数据")
            return NodeStatus.SUCCESS
        print("[BT] 图谱没有查到相关数据")
        return NodeStatus.FAIL


class CheckRetrievalConfidence(BTNode):
    """叶子: 校验检索结果置信度, 抑制幻觉"""
    def tick(self):
        print("[BT] 校验图谱结果置信度，交叉校验")
        mock_confidence_ok = True
        if mock_confidence_ok:
            return NodeStatus.SUCCESS
        return NodeStatus.FAIL


class LlmGenExplainAnswer(BTNode):
    """叶子: 基于图谱结果，生成可解释回答"""
    def __init__(self, query):
        self.query = query

    def tick(self):
        print(f"[BT] 大模型基于图谱链路生成可解释回答，问题：{self.query}")
        return NodeStatus.SUCCESS


class LlmFallbackAnswer(BTNode):
    """兜底叶子: 图谱查不到, 大模型直接泛回答 (不依赖图谱)"""
    def __init__(self, query):
        self.query = query

    def tick(self):
        print(f"[BT] 兜底分支：图谱检索失败，大模型直接泛回答：{self.query}")
        return NodeStatus.SUCCESS
```

**叶子节点的要点**：

- 每个叶子只做**一件明确的事**，职责单一；
- `mock_xxx = True` 是占位，真实项目里替换成你的 cypher 查询、向量检索、大模型调用；
- 叶子之间**不直接互相调用**，都由父节点（Sequence/Selector）编排，这就是「编排」的意义。

---

## 七、完整实现：图谱问答 Agent

把所有节点组装成一棵树，根节点是**选择节点**，含两条分支：

```
根节点 Selector
├─ 分支1 Sequence（优先走图谱链路）
│    ├─ ValidateQuery       校验提问
│    ├─ GraphRetrieve       图谱检索
│    ├─ CheckRetrievalConfidence  置信度校验
│    └─ LlmGenExplainAnswer 生成图谱回答
└─ 分支2 LlmFallbackAnswer  兜底：大模型直接泛回答
```

```python
if __name__ == "__main__":
    user_question = "大模型开发岗位需要掌握哪些前置技能？"

    # 组装整棵行为树: 根节点是选择器
    # 分支1 (优先走图谱链路): 校验问题 -> 图谱检索 -> 置信校验 -> 生成图谱回答
    # 分支2 (兜底)大模型直接回答
    agent_tree = Selector([
        Sequence([
            ValidateQuery(user_question),
            GraphRetrieve(user_question),
            CheckRetrievalConfidence(),
            LlmGenExplainAnswer(user_question)
        ]),
        LlmFallbackAnswer(user_question)
    ])

    print("===== 执行Agent行为树 =====")
    final_status = agent_tree.tick()
    print(f"\n行为树执行结果: {final_status.value}")
```

**这棵树读起来就是一段清晰的决策逻辑**：

1. 根 Selector 先尝试分支 1（Sequence）；
2. Sequence 依次执行「校验 → 检索 → 置信校验 → 生成」；
3. 只要 Sequence 里**任何一步失败**，整个分支失败，Selector 会转而尝试分支 2；
4. 分支 2 是大模型兜底回答，通常总能成功，保证 Agent 不会「什么都不答」。

> 这就是行为树最优雅的地方：**把「正常链路」和「兜底链路」的优先级、失败处理，全部表达成树结构，一眼看懂、易改易测。**

---

## 八、行为树速查表

| 点 | 要点 |
| --- | --- |
| 本质 | 树形结构 + 三态返回，编排 Agent 决策 |
| 三种节点 | Sequence / Selector / Leaf |
| Sequence | AND：依次执行，全成功才成功 |
| Selector | OR：挨个尝试，第一个成功就成功 |
| 叶子节点 | 执行真实动作，对接后端能力 |
| 三态 | SUCCESS / FAIL / RUNNING |
| 典型用法 | 正常链路走 Sequence，兜底走 Selector |
| 适用场景 | 游戏 AI、机器人、大模型 Agent |

---

**一句话总结**：行为树用「Sequence（AND）+ Selector（OR）+ 叶子动作」把 Agent 的决策流程组合成一棵树，正常链路、失败兜底、优先级都能清晰表达，是大模型 Agent 行为编排的优雅武器。

**文末预告**：算法系列到此收官，下一篇可以聊聊「AI-Agent 完整实战」——把检索、图谱、大模型、行为树串成一个能上线的智能体。

Let's use AI to create a better world。
