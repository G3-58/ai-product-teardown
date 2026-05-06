# 论文笔记｜Self-RAG: Learning to Retrieve, Generate, and Critique

> **论文**：Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection  
> **作者**：Asai et al.（University of Washington / Allen Institute for AI）  
> **发表**：NeurIPS 2023  
> **阅读时间**：2025.02  
> **关联项目**：ecom-competitor-agent 的检索质量优化 / 国网 Agent 的知识图谱 RAG 设计

---

## TL;DR

普通 RAG 是"每次都检索，不管有没有必要"。Self-RAG 让模型学会**自己判断**：这个问题需要检索吗？检索回来的内容有用吗？生成的答案是否有根据？通过在输出里插入 Reflection Token 来实现自我评估和选择性检索。

---

## 背景：标准 RAG 的问题

标准 RAG 流程（Naive RAG）：

```
Query → Retrieve Top-K → Concat → Generate
```

**问题 1 — 不必要的检索**：
"2+2等于几"不需要检索，但 Naive RAG 会检索，浪费 token，且检索结果可能引入噪音。

**问题 2 — 无关内容污染**：
检索到的文档可能与问题无关，但 LLM 还是会尝试用它们，导致幻觉。

**问题 3 — 没有质量评估**：
生成的答案是否真的基于检索内容？是否自相矛盾？标准 RAG 没有机制检查。

---

## Self-RAG 的核心机制

### Reflection Token

Self-RAG 训练模型输出特殊的 Reflection Token，插入到普通 token 流中：

| Token 类型 | 含义 | 可能值 |
|-----------|------|--------|
| `Retrieve` | 是否需要检索 | yes / no / continue |
| `ISREL` | 检索结果是否相关 | relevant / irrelevant |
| `ISSUP` | 生成内容是否有检索支撑 | fully / partially / no |
| `ISUSE` | 整体输出是否有用 | 1-5 分 |

### 完整推理流程

```
Query: "LangGraph 中 conditional_edge 怎么用？"

[Generate] → 需要检索吗？
→ Retrieve: yes

[Retrieve] → 调用检索器，获取 Top-K 文档

对每个文档 d_i：
  [Generate] ISREL(d_i) → relevant / irrelevant
  如果 relevant：
    [Generate] 基于 d_i 生成答案段落 y_i
    [Generate] ISSUP(y_i, d_i) → fully / partially / no
    [Generate] ISUSE(y_i) → 1-5

[Critique] 综合所有 (y_i, score_i)，选择或合并最优答案

[Output] 最终答案
```

### 为什么比 Naive RAG 强

1. **选择性检索**：简单问题不检索，复杂问题才检索，降低不必要的延迟和噪音
2. **文档筛选**：检索到 5 个文档，模型自己判断哪 2 个相关，不强制使用所有文档
3. **答案验证**：ISSUP token 检测"答案是否真的基于检索内容"，减少凭空捏造

---

## 与其他 RAG 改进的对比

| 方法 | 核心改进 | 缺点 |
|------|----------|------|
| **Naive RAG** | 基础 Retrieve-Generate | 噪音多，无法评估质量 |
| **HyDE** | 先生成假设文档再检索 | 检索改进，生成质量未改善 |
| **FLARE** | 迭代式检索，发现知识缺口时才检索 | 工程复杂，但无质量评估 |
| **Self-RAG** | 自我判断检索必要性 + 质量评估 | 需要训练（不能直接用 GPT-4 等黑盒模型） |
| **Corrective RAG（CRAG）** | 检索后评估相关性，低质量时用 Web 搜索补充 | 比 Self-RAG 更轻量，但无 ISSUP 类检查 |

**关键限制**：Self-RAG 需要特定的训练（微调一个开源模型来输出 Reflection Token），不能直接在 GPT-4 / Claude 上应用。

实际工程中，更可行的方案是用 Corrective RAG 的思路：用一个独立的"评估 LLM"来判断检索相关性，代替训练好的 Reflection Token。

---

## 联系工程实践

### ecom-competitor-agent 的检索质量问题

在电商竞对 Agent 里，最早遇到的问题就是 Naive RAG 的"噪音检索"：检索到了 5 个文档，但有 3 个其实和当前问题关系不大，导致 Writer Agent 输出中混入了不相关信息。

参考 Self-RAG 的思路，我加了一个"相关性过滤"步骤：

```python
# 伪代码
for doc in retrieved_docs:
    relevance = judge_llm.invoke(
        f"Document: {doc}\nQuery: {query}\nIs this document relevant? (yes/no)"
    )
    if relevance == "yes":
        filtered_docs.append(doc)
```

这个简单改动（用 LLM 过滤检索结果）把最终报告的信息准确性提升了明显。

### 国网 Agent 的知识图谱 RAG 改进方向

知识图谱 RAG 的特殊性：从图数据库检索出来的子图，可能包含大量相关但不直接有用的节点和关系。

Self-RAG 的 ISSUP 思想在这里很有参考价值：对检索到的图节点，可以加一步"是否对当前问题有贡献"的评估，只保留高贡献节点进入最终的 Context。

---

## 局限与反思

**Self-RAG 没有解决的问题**：

1. **检索本身的上限**：如果知识库里根本没有正确答案，自我评估再好也没用
2. **Reflection Token 的泛化性**：在训练集上表现好的 Reflection Token，在 OOD 问题上不一定可靠
3. **推理成本**：每次生成都要多输出 Reflection Token，且可能多次检索，实际延迟比 Naive RAG 高 2-3 倍

**产品取舍**：对延迟敏感的场景（客服实时对话），Self-RAG 太慢；对质量敏感的场景（法律/医疗/研究报告生成），值得用。

---

## 一句话总结

> Self-RAG 把 RAG 从"盲目检索 + 无脑生成"升级为"按需检索 + 自我审查"，核心贡献是把质量评估内化到生成过程里，而不是靠外部后处理。

---

## 关联阅读

- **CRAG（Corrective RAG）**（2024）：更轻量的替代方案，不需要训练，用评估 LLM 代替 Reflection Token
- **GraphRAG**（Microsoft，2024）：从文档构建知识图谱，解决 Naive RAG 无法处理全局问题的缺陷
- **RAG-Fusion**（2024）：多查询并行检索 + RRF 排序，提升检索召回率

---

*[原文链接](https://arxiv.org/abs/2310.11511)*
