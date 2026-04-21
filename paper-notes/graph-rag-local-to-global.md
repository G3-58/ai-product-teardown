# 论文笔记｜GraphRAG: From Local to Global

> **论文**：From Local to Global: A Graph RAG Approach to Query-Focused Summarization  
> **作者**：Edge et al.（Microsoft Research）  
> **发表**：2024.04（arXiv）  
> **阅读时间**：2025.01  
> **关联项目**：国网专利智能分析 Agent — 知识图谱检索模块

---

## TL;DR

传统 RAG 擅长回答「某个具体实体的信息」（local query），但对「这批文档讲了哪些主题」（global query）几乎无能为力——因为答案散落在所有 chunk 里，向量检索只取 top-K，天然有盲区。GraphRAG 通过构建知识图谱 + 层次化社区摘要，让 LLM 能回答跨文档的宏观问题。

---

## 核心思想

### 问题设定

给定一个大文档集合（例如所有专利文献），回答以下两类问题：
- **Local**：「张三发明人的主要技术方向是什么？」→ 传统 RAG 能搞定
- **Global**：「整个知识库中，哪些技术方向正在快速演进？」→ 传统 RAG 抓瞎

### 方法流程

```
文档集合
  ↓  (1) Entity & Relation Extraction（LLM抽取）
知识图谱 (节点=实体, 边=关系)
  ↓  (2) Community Detection（Leiden算法）
层次化社区（C0 最细粒度 → C3 最粗粒度）
  ↓  (3) Community Summary Generation
每个社区有一份 LLM 生成的摘要
  ↓  (4) Query-time
Local query → 图遍历 + 向量检索
Global query → 遍历社区摘要 → Map-Reduce 汇总
```

### 为什么用 Leiden 算法

Louvain 的改进版，模块度优化更稳定，不会陷入局部最优。在大规模图上社区划分质量更高。

---

## 我在国网项目里的实践

**场景**：数万条专利文献，需要回答「这个技术领域的发展脉络」「哪些发明人的技术路线相似」等 global 问题。

**我的实现**（简化版）：
- 实体抽取：用 Qwen + Prompt 提取「技术术语、发明人、IPC 分类」三类实体
- 图存储：Neo4j，节点用向量索引，边存关系权重
- 检索策略：本地问题走 BM25 + 向量混合召回；跨文档总结问题走社区摘要

**与论文的差异**：
- 没有做层次化社区（只有一层），因为专利语料结构相对规整，一层已经够用
- Community summary 是离线预生成的，查询时直接拼入 context，延迟更低

**效果**：Recall@10 从 62% → 87%，多跳问题（「A 技术和 B 技术有什么关联」）的回答质量提升最明显。

---

## 批判性思考

- **成本问题**：图构建阶段需要大量 LLM 调用（每个 chunk 都要抽实体），对于超大语料库成本很高
- **图质量依赖 LLM**：实体抽取的精度直接影响图的质量，领域术语识别需要微调或详细的 few-shot
- **更新问题**：新文档加入时如何增量更新图而不重建？论文没有解决，工程上是个难点

---

## 延伸阅读

- HippoRAG（2024）：用知识图谱模拟人类海马体记忆机制，解决图 RAG 的增量更新问题
- LightRAG（2024）：更轻量的图 RAG 实现，构建成本更低

---

## 一句话总结

> GraphRAG 是把「文档集合」变成「可推理的知识网络」，核心价值在 global 查询，但构建成本高，适合语料相对稳定、查询质量要求高的场景。

---

*[原文链接](https://arxiv.org/abs/2404.16130)*
