# AI 产品拆解 & 技术阅读笔记

> 系统性拆解主流 AI 产品 + 持续追踪 Agent / 空间智能等前沿方向。  
> 分三个栏目：产品拆解 / 论文笔记 / 技术博客阅读。

---

## 📱 产品拆解（teardowns/）

从用户体验、产品架构、Prompt 策略、商业模式四个维度系统拆解 AI 原生产品。

| 产品 | 公司 | 核心定位 | 拆解文档 |
|---|---|---|---|
| 豆包 | 字节跳动 | 泛化 AI 助手 / 内容消费入口 | [teardowns/doubao.md](teardowns/doubao.md) |
| Kimi | 月之暗面 | 长文档处理 / 深度搜索 | [teardowns/kimi.md](teardowns/kimi.md) |
| 通义千问 | 阿里云 | 企业级 AI / 生态整合 | [teardowns/tongyi.md](teardowns/tongyi.md) |

**拆解框架**：产品定位与用户 → 核心功能地图 → 交互设计分析 → Prompt 工程观察 → 能力边界测试 → 商业模式 → 竞品差异 → 个人感悟

待拆解：[ ] Perplexity  [ ] Claude.ai  [ ] Cursor  [ ] Notion AI

---

## 📄 论文笔记（paper-notes/）

精读与自己项目强相关的论文，重点记录工程实践和批判性思考，不只是摘要。

| 论文 | 关联项目 | 笔记 |
|---|---|---|
| DPO: Direct Preference Optimization（NeurIPS 2023）| Qwen2 偏好对齐实验 | [paper-notes/dpo-direct-preference-optimization.md](paper-notes/dpo-direct-preference-optimization.md) |
| GraphRAG: From Local to Global（Microsoft Research 2024）| 国网专利 Agent 知识图谱模块 | [paper-notes/graph-rag-local-to-global.md](paper-notes/graph-rag-local-to-global.md) |
| ReAct: Synergizing Reasoning and Acting（ICLR 2023）| LangGraph Agent 系统设计 | [paper-notes/react-reasoning-and-acting.md](paper-notes/react-reasoning-and-acting.md) |

---

## 📝 技术博客笔记（blog-notes/）

追踪 Agentic AI、空间智能等前沿方向的高质量博客，提炼对产品设计的启发。

| 文章 | 来源 | 核心观点 | 笔记 |
|---|---|---|---|
| Building Effective Agents | Anthropic（2024.12）| Agent ≠ 越自动越好；最小足迹原则 | [blog-notes/anthropic-building-effective-agents.md](blog-notes/anthropic-building-effective-agents.md) |
| LLM Powered Autonomous Agents | Lilian Weng（2023.06）| Agent = 规划 + 记忆 + 工具三件套 | [blog-notes/lilian-weng-llm-powered-agents.md](blog-notes/lilian-weng-llm-powered-agents.md) |
| 空间智能前沿追踪 | World Labs / 3DGS / CVPR | 下一代交互范式：从屏幕到空间 | [blog-notes/spatial-intelligence-frontier.md](blog-notes/spatial-intelligence-frontier.md) |
| Agentic AI 设计模式 | Andrew Ng / LangGraph / Devin | 5 种核心 Agent 模式 + 工程踩坑 | [blog-notes/agentic-ai-design-patterns.md](blog-notes/agentic-ai-design-patterns.md) |

---

## 关于这个仓库

在读研究生，AI 方向，同时关注 AI 产品设计与落地。  
这里是我学习过程中的思考沉淀——既有工程实践视角，也有产品设计视角。  

相关项目：[ai-spatial-workspace](https://github.com/G3-58/ai-spatial-workspace) · [multi-agent-report](https://github.com/G3-58/multi-agent-report) · [ecom-competitor-agent](https://github.com/G3-58/ecom-competitor-agent)
