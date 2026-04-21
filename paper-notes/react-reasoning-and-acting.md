# 论文笔记｜ReAct: Synergizing Reasoning and Acting

> **论文**：ReAct: Synergizing Reasoning and Acting in Language Models  
> **作者**：Yao et al.（Princeton / Google Brain）  
> **发表**：ICLR 2023  
> **阅读时间**：2024.12  
> **关联项目**：LangGraph Agent 系统设计的理论基础

---

## TL;DR

让 LLM 在执行任务时**交替输出 Thought 和 Action**——先想再做，根据执行结果更新推理，再决定下一步。这个简单的模式解决了纯 CoT（只推理不行动）和纯 Act（只调用工具不推理）的各自局限。

---

## 核心思想

### 三种范式对比

| 范式 | 做什么 | 缺陷 |
|------|--------|------|
| **CoT** | 只输出推理链，最后给答案 | 幻觉无法纠正，没有外部信息 |
| **Act-only** | 直接调用工具序列 | 没有推理，遇到复杂情况不会调整计划 |
| **ReAct** | Thought → Action → Observation → Thought → ... | 推理驱动行动，观察结果更新推理 |

### ReAct 的输出格式

```
Thought: 我需要先查找 X 的定义，再判断它是否满足条件 Y
Action: Search[X]
Observation: X 是...
Thought: 根据观察，X 满足 Y，但还需要确认 Z
Action: Lookup[Z]
Observation: Z 的值是...
Thought: 现在信息足够了，答案是...
Action: Finish[答案]
```

关键：**Thought 不是输出给用户的，是模型的「内心独白」**，但正是这个独白让模型能够修正错误、应对意外。

### 为什么有效

- Thought 提供了「可解释的推理轨迹」，出错时能 debug
- Observation 把外部环境的反馈注入推理链，避免幻觉堆叠
- 两者交替创造了「感知-规划-行动」的闭环，类似人类做任务的实际方式

---

## 联系 LangGraph 的工程实践

ReAct 是 Agent 系统的理论原型，LangGraph 是工程实现：

```
ReAct 的概念          LangGraph 的实现
─────────────────────────────────────────
Thought              → LLM node 的推理输出（往往是 tool_call 的 reasoning）
Action               → ToolNode 的执行
Observation          → ToolMessage 注入 state
循环控制              → conditional_edge + should_continue 函数
```

在我的国网 Agent 项目里：
- **检索 Agent** 的 Thought 是「这个问题需要先查技术分类，再找相关专利」
- Action 是调用 Neo4j 查询工具 / 向量检索工具
- Observation 是返回的图节点 / 文档片段
- 循环直到 Thought 判断「信息已足够生成报告」

**一个踩坑**：ReAct 对 Prompt 格式非常敏感。Thought/Action/Observation 的标记格式如果不统一，模型会混淆推理和工具调用。我最后用 LangChain 的 `create_react_agent` 接管了 prompt 模板，自己只写工具描述。

---

## 局限性

- **推理链很长**：token 消耗大，latency 高。每轮 Thought+Action+Observation 都要占 context，复杂任务容易超长
- **容易陷入循环**：如果工具返回的 Observation 模糊，模型可能重复调同一个工具。需要设计 max_steps 防护
- **单线程**：ReAct 是顺序执行的，不能并行调用多个工具。后来的 Plan-and-Execute、多 Agent 并行是对这个问题的改进

---

## 延伸阅读

- **Reflexion**（2023）：在 ReAct 基础上加了「失败后反思」，用语言描述错误原因，下一次尝试时避免重蹈
- **Toolformer**（2023）：让模型自己学什么时候该调工具，而不是靠 prompt 规定
- **LangGraph**：把 ReAct 的控制流固化成有向图，让复杂 Agent 可以工程化维护

---

## 一句话总结

> ReAct = Chain-of-Thought + Tool Use 的结合体，用「先想后做、看结果再想」的循环解决了 LLM 的幻觉和信息孤立问题，是现代 Agent 系统的思想原型。

---

*[原文链接](https://arxiv.org/abs/2210.03629)*
