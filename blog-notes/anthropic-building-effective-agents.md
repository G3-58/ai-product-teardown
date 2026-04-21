# 博客笔记｜Building Effective Agents — Anthropic

> **来源**：Anthropic Engineering Blog  
> **原文**：https://www.anthropic.com/research/building-effective-agents  
> **发布**：2024.12  
> **阅读时间**：2025.01  
> **一句话**：这篇是目前工程视角最实用的 Agent 设计指南，没有之一。

---

## 为什么值得读

大多数 Agent 教程教你怎么接 API，这篇教你**什么时候不该用 Agent**。Anthropic 用自己的工程实践总结了 Agent 的适用边界和反模式，读完之后对「什么是好的 Agent 产品设计」有清晰很多的直觉。

---

## 核心框架

### Workflow vs Agent 的区分

Anthropic 把自动化系统分两类：

| 类型 | 特征 | 适用场景 |
|------|------|---------|
| **Workflow** | 预定义控制流，LLM 只填充局部 | 步骤固定、可预测的任务 |
| **Agent** | LLM 自主决定步骤和工具调用序列 | 步骤不可预测、需要复杂判断 |

**关键建议**：优先用 Workflow，只有当任务的步骤序列确实无法预先设计时才升级到 Agent。大多数产品过度使用了 Agent。

### 五种 Workflow 模式

**1. Prompt Chaining（提示链）**  
把复杂任务拆成顺序步骤，每步输出是下步输入。  
适合：写作 → 审查 → 格式化 这类有明确顺序的任务。

**2. Routing（路由）**  
根据输入分类，分发到不同的专化处理流程。  
适合：客服系统里区分「退款」「投诉」「咨询」。

**3. Parallelization（并行化）**  
多个子任务同时跑，结果聚合。  
适合：多文档同时分析，最后汇总。

**4. Orchestrator-Subagents（主控-子 Agent）**  
一个 Orchestrator 规划任务，动态派发给专化的子 Agent。  
适合：复杂研究任务、代码生成+测试+修复的循环。

**5. Evaluator-Optimizer（评估-优化循环）**  
一个 LLM 生成，另一个评估，不满足就打回重来。  
适合：写作质量要求高、需要自我改进的场景（我的 multi-agent-report 项目用的就是这个模式）。

### Agent 设计原则

**Minimal Footprint（最小足迹）原则**：  
Agent 应该只申请完成任务必须的权限，优先可逆操作，关键节点要有确认机制。  
→ 这直接影响用户信任感。一个 Agent 如果动不动就有大量权限，用户很快会不敢用。

**人工介入点的设计**：  
不是所有决策都应该自动化。文章建议在以下情况停下来问用户：
- 不可逆操作前（删除、发送）
- 置信度低的分类决策
- 涉及金钱或重要数据

---

## 我的思考

**最触动我的一点**：

Anthropic 说「大多数实际问题不需要 Agent，Workflow 足够了」。这对我影响很大——我在国网项目里最开始设计了一个五步 Agent，后来发现其中三步的顺序是固定的，改成 Workflow + 单步 Agent 之后，系统稳定性提升了很多，debug 也容易多了。

**产品设计的启发**：

「用户愿意把控制权交给 Agent」这件事不是理所当然的。Anthropic 的 minimal footprint 原则本质上是在说：Agent 的自主性需要通过一次次可预期的行为来赢得。产品上要设计「可观察性」（用户能看到 Agent 在做什么）和「可干预点」（用户能随时叫停）。

这个问题在我的电商竞对 Agent 里也踩过——最开始是全自动生成报告然后直接发送，运营同学不信任，改成「生成草稿 → 人工确认 → 发送」之后接受度高很多。

---

## 一句话总结

> Agent ≠ 越自动越好。好的 Agent 产品设计是：用最简单的控制流解决问题，在用户信任建立之前保持「可观察、可干预、最小权限」。

---

*[原文链接](https://www.anthropic.com/research/building-effective-agents)*
