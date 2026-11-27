---
title: "AgentAI开发前置知识 "
status: 已发布
author: XiaoYi
---

# AgentAI开发前置知识 

> 2026 年最热的技术岗是什么？AI Agent 开发。大厂都在抢「能搭智能体、会做 RAG、懂路由优化、懂多智能体协作」的人。
> 这一篇把你进大厂需要的 Agent 必备知识全景一次讲透：核心架构、提示词工程、路由系统、RAG、多智能体、安全护栏、开发工具链。学完这一篇，你只要再补上 LangChain / LangGraph 两个框架，就够进大厂干活了。
> Let's use AI to create a better world。

## 一、Agent 是什么（核心认知）

**智能体（Agent）** 的定义，一句话：

> **一个能自己制定计划、自己动手干活、干错了能自我修改的软件系统。**

**核心公式（面试必背）**：

```
Agent = Model + Harness
```

|  |  |  |
| --- | --- | --- |
| 部分 | 含义 | 类比 |
| **Model** | 大模型（大脑） | 思考能力 |
| **Harness** | 外壳/脚手架（手、脚、工具） | 执行能力 |

光有大模型只会「说」，加上 Harness（工具调用、记忆、循环执行、自我纠错）才能「做」。

**MCP = Model Context Protocol**，是 AI 世界的 **"USB 接口"**——统一了「模型怎么连接各种工具/数据」的标准，让不同模型能即插即用地接入各种能力。

**AI Agent 完整公式**：

```
AI Agent = 大模型 + 记忆 + 任务规划 + 工具使用 + 行动
```

---

## 二、Agent 五大构成模块

|  |  |  |
| --- | --- | --- |
| 模块 | 作用 | 例子 |
| **感知模块** | 接收用户提问、环境信息、上下文 | 解析输入 |
| **记忆模块** | 短期会话记忆 + 长期知识库记忆 | 对话历史、向量库 |
| **规划推理模块** | 思考分析、任务拆解、逻辑推理 | 把「帮我做 PPT」拆成 5 步 |
| **工具调用模块** | 调用接口/数据库、第三方能力、大模型 | 查数据库、调 API |
| **行动输出模块** | 整理答案回复、执行具体业务动作 | 返回结果、执行操作 |

> 记忆：**感知（听）→ 规划（想）→ 工具（做）→ 行动（答），全程靠记忆（记）串联。**

---

## 三、Bot vs Agent

这是面试高频「区别题」：

|  |  |  |
| --- | --- | --- |
| 对比 | Bot | Agent |
| 响应方式 | 被动响应指令 | 主动制定计划 |
| 自主决策 | ❌ 无 | ✅ 有 |
| 行为模式 | 严格按规则/指令执行 | 能自我纠错、动态调整 |
| 适用 | 固定流程、简单自动化 | 复杂任务、需要推理 |

> 一句话：**Bot 是「照着剧本演」，Agent 是「看着情况临场发挥」。**

---

## 四、提示词工程

### 五大原则

|  |  |
| --- | --- |
| 原则 | 说明 |
| 1. 清晰的指令 | 明确告诉模型「做什么、怎么做」 |
| 2. 文本参考 | 给足背景资料/示例 |
| 3. 复杂任务拆分成子任务 | 大任务拆小步骤 |
| 4. 给模型「思考」时间 | 让它一步步推理（CoT 思维链） |
| 5. 借助外部工具 | 算不来的交给工具（代码执行、搜索） |

### 提示词规范（六要素）

```
1. 角色  2. 任务  3. 工具  4. 规则  5. 限制  6. 样例
```

### Few-Shot（少样本提示）

给几个「输入 → 输出」的例子，让模型照着格式回答，比纯描述更稳定。

> 记忆：**提示词写得好，Agent 能发挥 80 分；写得烂，再强的模型也只有 40 分。**

---

## 五、路由系统（重点）

路由是 Agent 进大厂的**核心进阶能力**——核心思想是：**不同任务交给最合适的处理对象，降成本、提速度、保质量。**

### 5.1 负载均衡

同类服务器有「忙碌 / 空闲 / 较忙」之分，把请求发给**空闲**的那台，避免单点过载。

### 5.2 动态路由

判断任务该交给**哪一类**处理对象：

```
任务 → 小模型 / 强模型 / RAG / 数据库 / 工具 / Agent
```

### 5.3 Agent 条件路由

用**条件判断**决定执行策略（if-else 逻辑）：比如「是简单问答走 A，是复杂任务走 B」。

### 5.4 模型分级路由（省钱关键）

```
先试用简单模型 → 检查结果是否符合标准
   ├─ 符合 → 直接用（便宜、快）
   └─ 不符合 → 升级到更高级模型重新处理
```

> 思路：**大部分简单问题用便宜的小模型搞定，少数难题才动用贵的大模型**，平均成本大幅下降。

### 5.5 FallBack（兜底）

主模型处理失败时，切换到**备用方案**（备用模型 / 降级逻辑），保证服务不中断。

### 5.6 常见四类路由

|  |  |
| --- | --- |
| 路由类型 | 作用 |
| 模型路由 | 不同任务选不同模型 |
| 工具路由 | 不同任务选不同工具 |
| RAG 路由 | 是否走检索、走哪个知识库 |
| 工作流路由 | 选哪条执行流程 |

> 一句话总结路由目标：**降低平均成本、提高响应速度、保持准确率、复杂问题可处理。**

---

## 六、LLM 选择策略

选哪个模型，不是拍脑袋，要看这些维度：

|  |  |
| --- | --- |
| 维度 | 说明 |
| 问题长度 | 长文本 vs 短问答 |
| 任务类型 | 生成 / 分类 / 摘要 / 代码 |
| 是否含代码/图片 | 多模态需求 |
| Token 量 | 输入输出规模 |
| 任务复杂度 | 简单 vs 复杂推理 |
| 延迟要求 | 实时 vs 离线 |
| 预算 | 成本敏感度 |
| 历史失败次数 | 失败多就升级模型 |
| 用户等级 | VIP 用强模型 |

### 三级筛选（逐级递进）

```
1. 规则（Rule）      → 最简单，关键词/规则直接命中
2. Embedding（向量） → 语义相似度匹配
3. LLM              → 前两级搞不定的，交给大模型兜底
```

> 思路：**能规则解决的别用向量，能向量解决的别用大模型**——每一级都在省钱提速。

---

## 七、RAG 搭建流程

RAG（检索增强生成）是让大模型「回答自己的私有数据」的核心方案。搭建流程 8 步：

|  |  |
| --- | --- |
| 步骤 | 内容 |
| 1 | 需求分析：知识范围、数据源、技术选型、架构设计 |
| 2 | 数据收集与预处理 / 清洗 |
| 3 | 文本切分与向量化（chunk + embedding） |
| 4 | 向量数据库搭建 |
| 5 | 检索模块开发 |
| 6 | 生成与回答模块 |
| 7 | 评估与优化 |
| 8 | 部署与运维 |

**主流向量库**：Chroma（轻量）、Milvus（生产级分布式）、FAISS、Pinecone、Weaviate 等。

> RAG 的核心价值：**让大模型能「查」到私有知识，而不是只靠训练时的记忆。**

---

## 八、Agent 搭建流程

9 步标准流程：

|  |  |
| --- | --- |
| 步骤 | 内容 |
| 1 | 需求分析与项目架构 |
| 2 | 定义 Agent 与实现配置 |
| 3 | 集成工具（Tool） |
| 4 | 开发技能 Skills |
| 5 | 接入记忆 |
| 6 | 设计提示词 |
| 7 | 测试与调试 |
| 8 | 部署与运行 |
| 9 | 监控与优化 |

**语音技术补充**：ASR（语音识别，把声音转文字）+ TTS（语音合成，把文字转声音），是语音 Agent 的两大基础能力。

---

## 九、多智能体协作（A2A / MCP / SKILL）

单 Agent 能力有限，复杂项目需要**多智能体分工干活**。

|  |  |  |
| --- | --- | --- |
| 概念 | 含义 | 类比 |
| **A2A** | Agent To Agent 协议，多智能体协作标准 | 智能体之间的「对话协议」 |
| **MCP** | 模型连接工具/数据的标准 | AI 世界的 USB 接口 |
| **SKILL** | 适合工作流定义，软件 Skills Manager 管理 | 可复用的「技能包」 |

**分工逻辑**：

```
MCP 功能 = 增强扩展（给 Agent 加「手」）
SKILL 方式 = 适合工作流定义（给 Agent 加「技能套路」）
A2A = 多智能体之间协作（让多个 Agent 组队）
```

> 趋势：**多智能体 + 工作流（Skill）+ 工具（MCP）** 是当前 Agent 落地的三大支柱。

---

## 十、评测框架 + 安全护栏

### 评测框架

**如何量化 AI 的表现**——用指标衡量 Agent 好不好，而不是凭感觉：

- 准确率、召回率（RAG 检索质量）

- 响应速度、延迟

- 成本

- 任务完成率、失败率

- 用户满意度

### 安全护栏（进大厂必考）

**如何防止 AI 说出不该说的话、做不该做的事**：

|  |  |
| --- | --- |
| 护栏类型 | 作用 |
| 输入过滤 | 拦截敏感/违规输入 |
| 输出审查 | 检查生成内容是否合规 |
| 权限控制 | 限制工具调用的范围 |
| 人工审核 | 高风险操作人工确认 |

> ⚠️ **风险认知**：存在「越狱（Jailbreak）」——攻击者通过精心设计的提示词诱导，试图绕过模型的安全机制。作为 Agent 开发者，你必须了解这个风险，并**用多层安全护栏 + 权限最小化 + 敏感操作人工确认**来防御。**安全是 Agent 上线的底线**，不是可选项。

---

## 十一、开发工具链

进大厂干活，工具链得溜。你笔记里的全套整理如下：

### Cursor（推荐 Agent 开发编辑器）

|  |  |
| --- | --- |
| 快捷键（Windows） | 功能 |
| `Ctrl + L` | 打开侧边聊天面板 |
| `Ctrl + /` | 循环切换模型 |
| `Ctrl + Shift + L` | 把选中代码追加进当前聊天上下文 |
| `Tab` | 接受 AI 自动补全整段 |
| `Ctrl + →` | 只接收自动补全的下一个单词 |
| `Ctrl + Enter` | 接受 AI 所有修改 |
| `Ctrl + Backspace` | 拒绝 AI 所有修改 |
| `Ctrl + N` | 新建对话 |
| `Ctrl + [` / `Ctrl + ]` | 上一个 / 下一个对话 |
| `Ctrl + P` | 快速跳转文件 |
| `Ctrl + Shift + P` | 命令面板（改主题等） |
| `F12` | 跳转定义/引用 |
| `Ctrl + G` | 跳转到指定行号 |
| `Ctrl + Shift + O` | 查找函数/变量符号 |
| `Ctrl + B` | 开关左侧文件树 |
| `Ctrl + ~` | 召出终端 |

### 关键插件

|  |  |
| --- | --- |
| 插件 | 作用 |
| **cline** | 开源 Harness，支持各种 IDE，`Ctrl + '` 快速引用选中内容 |
| **continue** | AI 自动代码补全 |
| **Cookie-Editor** | 浏览器抓取 Cookies 工具 |
| X-LocalAutoCompletion / WorkPeek | 单词提示插件 |

### continue 配置示例

```
name: Main Config
version: 1.0.0
schema: v1
models:
  - name: TuoTuoMiao
    provider: openai
    model: Deepseek-V4-Pro
    apiBase: https://xxx.xxx/v1
    apiKey: <apiKey>
    roles:
      - autocomplete
      - chat
```

> 配置技巧：符合 OpenAI 规范的中转站 API，Provider 选 **OpenAI Compatible** 即可对接。

### VSCode 前端插件清单

Markdown Preview Enhanced、Markdown Preview Mermaid Support、SCSS IntelliSense、Auto Rename Tag、Path Intellisense、DotENV、GitLens、Better Comments、Better Todo Tree、CSS Peek、ESLint、Live Server、HTMLHint、Volar/Vue-Official（Vue3）、Prettier、Error Lens。

### VSCode settings.json（前端）

```
{
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": "explicit"
    },
    "vue.complete.casing.tags": "auto",
    "path-intellisense.mappings": {
        "@": "${workspaceFolder}/src"
    },
    "prettier.semi": false,
    "prettier.singleQuote": true
}
```

### Python 插件 + 配置

|  |  |
| --- | --- |
| 插件 | 作用 |
| Python | 语法识别、解释器选择、基础调试 |
| Pylance | 类型提示、自动导入、查找引用、变量追踪 |
| Black Formatter | 代码格式化 |

```
{
    "editor.formatOnSave": true,
    "[python]": {
        "editor.defaultFormatter": "ms-python.black-formatter"
    },
    "python.analysis.typeCheckingMode": "basic",
    "python.analysis.autoImportCompletions": true,
    "python.analysis.importFormat": "relative",
    "python.analysis.diagnosticMode": "workspace",
    "python.terminal.activateEnvironment": true
}
```

### pyproject.toml（Black 配置）

```
[tool.black]
skip-string-normalization = true  # 不自动把双引号改成单引号
```

### Git 改动标记（settings.json）

```
{
    "scm.diffDecorations": "gutter",
    "scm.diffDecorationsGutterVisibility": "always",
    "scm.diffDecorationsGutterWidth": 4
}
```

### IDE 快捷键对比

|  |  |  |
| --- | --- | --- |
| 操作 | VSCode | Pycharm/Idea |
| 快速打开文件 | `Ctrl + P` | `Ctrl + Shift + N` |
| 命令面板 | `Ctrl + Shift + P` | — |
| 快速修复 | — | `Alt + Enter` |

### Pycharm / Idea 插件

CodeGlance Pro（代码缩略图）、Indent Rainbow（缩进彩虹）、Inspection Lens、Rainbow Brackets Lite（彩色括号）。

### 实用小技巧

- `Alt + 鼠标竖直滚轮`：多行光标操作

- `Ctrl + F`：搜索

- `Ctrl + 点击`：查看源码

**推荐工具：**

**codebuddy 类似 cursor**

**workbuddy 多agent工作台**

**claudecode 命令/界面agent-harness**

**codex 国际知名模型harness**

---

## 十二、工程化模板（角色定义）

你笔记最后的「角色定义」模板，是**写 Agent 提示词的工程化骨架**，直接套用：

```
# 角色定义
（你是谁、做什么）

# 特殊技能
（你擅长什么）

# 执行步骤
（一步步怎么做）

# 输出规范
（结果怎么输出）

# 约束黑名单
（什么不能做）

# 验收条件
（做到什么算完成）
```

> 这套模板把你的「提示词六要素」（角色/任务/工具/规则/限制/样例）结构化，让 Agent 行为可控、可验收。

---

## 十三、进大厂知识地图

**你已经掌握的（本篇覆盖）**：

- ✅ Agent 核心架构（五大模块、Model + Harness）

- ✅ 提示词工程（五大原则、六要素、Few-Shot）

- ✅ 路由系统（负载均衡、动态/条件/分级路由、FallBack）

- ✅ LLM 选择 + 三级筛选

- ✅ RAG 全流程

- ✅ 多智能体（A2A / MCP / SKILL）

- ✅ 评测 + 安全护栏

- ✅ 工具链（Cursor / cline / continue / VSCode）

**还需补充（下一步）**：

```
LangChain  → 统一的 Agent 开发框架（封装了模型调用、工具、记忆、链）
LangGraph  → 状态图编排（把 Agent 做成可控的状态机，支持多智能体）
```

> 学完本篇 + LangChain + LangGraph，你就具备了大厂 Agent 岗的**完整能力栈**，可以独立搭建生产级智能体。

---

## 速查表

|  |  |
| --- | --- |
| 需求 | 答案 |
| Agent 核心公式 | Agent = Model + Harness |
| Agent 五大模块 | 感知 / 记忆 / 规划 / 工具 / 行动 |
| MCP 是啥 | AI 世界的 USB 接口（模型连工具的标准） |
| Bot vs Agent | Bot 被动执行，Agent 自主决策 |
| 提示词六要素 | 角色/任务/工具/规则/限制/样例 |
| 省钱核心 | 模型分级路由 + 三级筛选 |
| RAG 是啥 | 检索增强生成，让模型查私有数据 |
| 多智能体三支柱 | A2A（协作）+ MCP（工具）+ SKILL（工作流） |
| 安全底线 | 多层安全护栏 + 权限最小化 + 人工确认 |
| 进大厂还要补啥 | LangChain + LangGraph |
