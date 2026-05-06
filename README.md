# AI 产品拆解 & 技术阅读笔记

> 系统性拆解主流 AI 产品 + 持续追踪 Agent / RAG / 推理模型等前沿方向。  
> 分三个栏目：产品拆解 / 论文笔记 / 技术博客阅读。

---

## 📱 产品拆解（teardowns/）

从用户体验、产品架构、Prompt 策略、商业模式四个维度系统拆解 AI 原生产品。

| 产品 | 公司 | 核心定位 | 拆解文档 |
|------|------|----------|----------|
| Cursor | Anysphere | AI-native 代码编辑器 / 工程效率 | [teardowns/cursor.md](teardowns/cursor.md) |
| 豆包 | 字节跳动 | 泛化 AI 助手 / 内容消费入口 | [teardowns/doubao.md](teardowns/doubao.md) |
| Kimi | 月之暗面 | 长文档处理 / 深度搜索 | [teardowns/kimi.md](teardowns/kimi.md) |
| 通义千问 | 阿里云 | 企业级 AI / 阿里生态整合 | [teardowns/tongyi.md](teardowns/tongyi.md) |
| Perplexity | Perplexity AI | 答案引擎 / 引用优先 / Deep Research | [teardowns/perplexity.md](teardowns/perplexity.md) |
| Notion AI | Notion | 工具原生 AI / 知识库问答 | [teardowns/notionai.md](teardowns/notionai.md) |
| Claude | Anthropic | 可信赖 AI / Extended Thinking | [teardowns/claude.md](teardowns/claude.md) |

**拆解框架**：产品定位与用户 → 核心功能地图 → 交互设计分析 → Prompt 工程观察 → 能力边界测试 → 商业模式 → 竞品差异 → 个人感悟

---

## 📄 论文笔记（paper-notes/）

精读与自己项目强相关的论文，重点记录工程实践和批判性思考，不只是摘要。

| 论文 | 核心贡献 | 关联项目 | 笔记 |
|------|----------|----------|------|
| ReAct: Synergizing Reasoning and Acting（ICLR 2023）| Thought-Action-Observation 循环，现代 Agent 的思想原型 | LangGraph Agent 系统 | [paper-notes/react-reasoning-and-acting.md](paper-notes/react-reasoning-and-acting.md) |
| Chain-of-Thought Prompting（NeurIPS 2022）| 让模型把推理步骤说出来，显著提升复杂推理能力 | ai-prd-gen Agent Prompt 设计 | [paper-notes/chain-of-thought-prompting.md](paper-notes/chain-of-thought-prompting.md) |
| Self-RAG（NeurIPS 2023）| 选择性检索 + 自我评估，解决 Naive RAG 的噪音和质量问题 | ecom-competitor-agent 检索优化 | [paper-notes/self-rag.md](paper-notes/self-rag.md) |
| GraphRAG: From Local to Global（Microsoft Research 2024）| 知识图谱 RAG，解决全局性问题 | 国网专利 Agent 知识图谱模块 | [paper-notes/graph-rag-local-to-global.md](paper-notes/graph-rag-local-to-global.md) |
| DPO: Direct Preference Optimization（NeurIPS 2023）| 更简单的 RLHF 替代方案，直接优化偏好 | Qwen2 偏好对齐实验 | [paper-notes/dpo-direct-preference-optimization.md](paper-notes/dpo-direct-preference-optimization.md) |

---

## 📝 技术博客笔记（blog-notes/）

追踪 Agentic AI、RAG、推理模型等前沿方向的高质量博客，提炼对产品和工程设计的启发。

| 文章 | 来源 | 核心观点 | 笔记 |
|------|------|----------|------|
| Building Effective Agents | Anthropic（2024.12）| Agent ≠ 越自动越好；最小足迹原则 | [blog-notes/anthropic-building-effective-agents.md](blog-notes/anthropic-building-effective-agents.md) |
| OpenAI o1 推理模型分析 | OpenAI System Card + 社区（2024.09）| 推理时计算 Scaling，改变产品分层逻辑 | [blog-notes/openai-o1-reasoning-model.md](blog-notes/openai-o1-reasoning-model.md) |
| RAG 技术演进：Naive → Advanced | LlamaIndex / LangChain / Pinecone（2024-2025）| Hybrid Search + Rerank + Agentic RAG 的工程最优实践 | [blog-notes/rag-from-naive-to-advanced.md](blog-notes/rag-from-naive-to-advanced.md) |
| LLM Powered Autonomous Agents | Lilian Weng（2023.06）| Agent = 规划 + 记忆 + 工具三件套 | [blog-notes/lilian-weng-llm-powered-agents.md](blog-notes/lilian-weng-llm-powered-agents.md) |
| Agentic AI 设计模式 | Andrew Ng / LangGraph / Devin | 5 种核心 Agent 模式 + 工程踩坑 | [blog-notes/agentic-ai-design-patterns.md](blog-notes/agentic-ai-design-patterns.md) |
| 空间智能前沿追踪 | World Labs / 3DGS / CVPR | 下一代交互范式：从屏幕到空间 | [blog-notes/spatial-intelligence-frontier.md](blog-notes/spatial-intelligence-frontier.md) |

---

## 关于这个仓库

在读研究生，AI 方向，同时关注 AI 产品设计与落地。  
这里是我学习过程中的思考沉淀——既有工程实践视角，也有产品设计视角。

相关项目：
[ai-prd-gen](https://github.com/G3-58/ai-prd-gen) · [prompt-lab](https://github.com/G3-58/prompt-lab) · [ai-spatial-workspace](https://github.com/G3-58/ai-spatial-workspace) · [multi-agent-report](https://github.com/G3-58/multi-agent-report) · [ecom-competitor-agent](https://github.com/G3-58/ecom-competitor-agent)
