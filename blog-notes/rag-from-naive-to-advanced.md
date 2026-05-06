# 博客笔记｜RAG 技术演进：从 Naive 到 Advanced

> **来源**：综合多篇博客与论文  
> 主要参考：LlamaIndex Blog / LangChain Blog / Pinecone RAG Guide / Weaviate Blog  
> **阅读时间**：2025.02-03（持续整理）  
> **一句话**：RAG 不是一个技术，而是一个系统设计问题；Naive RAG 只是起点。  
> **关联项目**：ecom-competitor-agent 的检索系统 / 国网专利 Agent 的知识图谱 RAG

---

## 为什么要系统学 RAG

RAG（检索增强生成）是目前 AI 工程里最"落地"的技术路线：不需要训练，直接用；可以实时更新知识库；可以提供来源引用，解决幻觉问题。

但"Naive RAG"在真实产品场景里的表现往往让人失望——召回率不高、生成内容偏题、latency 太大。理解 RAG 的演进路径，才能做出在实际业务中 work 的检索系统。

---

## RAG 的三代演进

### 第一代：Naive RAG

```
Query → Embedding → 向量相似度检索 Top-K → 拼接 Context → LLM 生成
```

**核心组件**：
- 文档切片（Chunking）：固定长度切分，通常 512-1024 token
- Embedding 模型：text-embedding-3-small / bge-large-zh 等
- 向量数据库：Pinecone / Chroma / Weaviate / FAISS

**典型问题**：
- 语义相似 ≠ 信息相关（问"怎么提高销售额"，检索出"销售额的定义"）
- 固定 Chunk 切断了上下文（一段代码被切断在两个 Chunk 里）
- 检索到的 Top-K 里可能有大量噪音
- 多跳问题（需要两个文档里的信息结合才能回答）完全失效

---

### 第二代：Advanced RAG

针对 Naive RAG 的具体问题，工程界发展出了多个改进方向：

#### 改进 1 — 更好的 Chunking 策略

| 策略 | 适用场景 |
|------|---------|
| 固定大小切分 | 最简单，通用 |
| 句子级切分 + 重叠 | 保留上下文连续性 |
| 语义切分（按段落/标题） | 结构化文档（PDF/Word） |
| Parent-Child 切分 | 小 Chunk 检索，大 Chunk 注入 Context |
| 文档摘要作为 Chunk | 先摘要后检索，提升语义密度 |

**Parent-Child 切分**（LlamaIndex 首创）：
- 小 Chunk（~128 token）用于精确匹配检索
- 检索到小 Chunk 后，返回其父 Chunk（~512 token）作为 Context
- 效果：检索精度和 Context 完整性两者兼顾

#### 改进 2 — Query 处理

**Query Rewriting**：把用户的模糊问题改写成更适合检索的形式。
```
原始 Query: "这个功能怎么用？"
改写后: "功能 X 的使用方法、步骤、注意事项"
```

**HyDE（Hypothetical Document Embeddings）**：先让 LLM 生成一个"假设的答案文档"，然后用这个假设文档的 embedding 去检索。

原理：答案文档和真实文档在语义空间里更接近，比 Query 本身更准。

**Multi-Query**：同一问题从多个角度生成 3-5 个不同的 Query，并行检索，合并去重结果。提升召回率，但 latency 增加。

#### 改进 3 — 混合检索（Hybrid Search）

单纯向量检索的弱点：精确词匹配（品牌名、专有名词、代码变量名）的效果比关键词检索差。

**BM25 + 向量检索 + Rerank**（目前工程最优实践）：

```
Query
├─ BM25 检索（关键词） → Top-K1 结果
└─ 向量检索（语义）   → Top-K2 结果
        ↓
    结果合并（RRF 融合排序）
        ↓
    Rerank 模型（cross-encoder）精排
        ↓
    Top-N 最终结果
```

**RRF（Reciprocal Rank Fusion）**：不关心两个检索结果的分数绝对值，只看排名，加权合并。公式简单，效果好。

**Rerank**：用 cross-encoder 模型（比 bi-encoder 更精准但更慢）对候选集做精排。常用：Cohere Rerank / bge-reranker-large。

这也是我在 ecom-competitor-agent 里用的方案，混合检索 + Rerank 比纯向量检索的相关性提升了约 20%（基于人工评估 50 条查询）。

#### 改进 4 — 检索后处理

**Contextual Compression**：检索到长文档后，先用 LLM 提取与问题相关的段落，再注入 Context，压缩无关信息。

**MMR（Maximal Marginal Relevance）**：在选 Top-K 时，不只选最相关的，还要选"与已选结果不重复"的——平衡相关性和多样性，避免 5 个结果说的是同一件事。

---

### 第三代：Modular RAG / Agentic RAG

RAG 系统开始变得模块化，可以动态组合不同的检索策略：

#### Graph RAG（微软，2024）

传统 RAG 处理不了"全局性问题"：
- "这份年报里整体的风险因素有哪些？"（需要综合全文）
- "这个产品线的核心竞争力是什么？"（需要跨文档整合）

Graph RAG 先从文档构建知识图谱（实体 + 关系），再从图上做检索：
- 局部问题 → 向量检索相关节点
- 全局问题 → 图遍历 + 社区摘要

代价：构建知识图谱消耗大量 LLM 调用（成本高），适合对质量要求极高的场景。

这与我在国网项目里做的知识图谱 RAG 方向一致——专利数据天然有实体关系（发明人/专利/技术分类/引用关系），图结构比向量检索更能表达"谁引用了谁"、"哪个技术方向最活跃"这类关系型查询。

#### Agentic RAG

让 Agent 自主决定检索策略：
- 判断是否需要检索（vs 直接回答）
- 判断检索结果是否足够（vs 继续检索）
- 判断从哪个数据源检索（内部文档 / 向量库 / 知识图谱 / 网络搜索）

这就是 Self-RAG（Asai et al.）和 Corrective RAG 的工程化实现。

---

## 实际工程中的选型建议

根据我在多个项目里的实践，整理了一个决策树：

```
需求场景
├── 文档量小（< 1000 文档），精确词匹配重要
│   → BM25 + 简单向量检索，不需要 Rerank
├── 文档量中等，语义理解重要
│   → 混合检索（BM25 + 向量）+ Rerank
├── 文档有明确结构（PDF/Word 标题层级）
│   → Parent-Child Chunking + 语义切分
├── 需要回答全局性/综合性问题
│   → Graph RAG
├── 实时性要求高（< 2s 响应）
│   → 纯向量检索（不做 Rerank），控制 Chunk 数量
└── 质量要求最高，latency 可接受
    → Agentic RAG（Self-RAG 或 Corrective RAG）
```

---

## 容易忽视的工程细节

1. **Embedding 模型的选择**：中文场景用 bge-large-zh 或 text-embedding-3-large（OpenAI），不要直接用英文 Embedding 模型处理中文
2. **Chunk 大小的调参**：不存在"最优 Chunk 大小"，需要根据文档类型和查询类型实验
3. **向量数据库的选型**：本地实验用 FAISS/Chroma；生产环境用 Qdrant（开源自部署）或 Pinecone（托管）
4. **索引更新策略**：文档更新后要重新 Embedding，这个成本在大型知识库里不可忽视
5. **评估指标**：RAG 的评估不能只看最终回答质量，要分开评估"检索质量"（Recall@K）和"生成质量"（faithfulness / relevance），才能定位瓶颈

---

## 一句话总结

> RAG 的进化方向是：从"查了再生成"到"按需查、智能选、有自知之明"。Naive RAG 适合 Demo，Advanced RAG 才适合产品。

---

## 关联阅读

- **Self-RAG**（NeurIPS 2023）：把 RAG 质量评估内化为模型行为
- **GraphRAG**（微软，2024）：解决 Naive RAG 的全局问题盲区
- **CRAG（Corrective RAG）**（2024）：检索质量不够时自动用 Web 搜索补充
- **RAG-Fusion**（2024）：多查询 + RRF 融合，提升召回率
