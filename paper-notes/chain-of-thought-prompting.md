# 论文笔记｜Chain-of-Thought Prompting Elicits Reasoning in LLMs

> **论文**：Chain-of-Thought Prompting Elicits Reasoning in Large Language Models  
> **作者**：Jason Wei, Xuezhi Wang, Dale Schuurmans et al.（Google Brain）  
> **发表**：NeurIPS 2022  
> **阅读时间**：2025.01  
> **关联项目**：ai-prd-gen 的 Agent Prompt 设计 / multi-agent-report 的 Synthesizer Agent

---

## TL;DR

在 Few-shot Prompt 的示例里，把"思考步骤"也写出来（而不只是"问题→答案"），就能显著提升 LLM 在复杂推理任务上的表现。这个简单的发现，重新定义了我们对 Prompt 工程的理解。

---

## 核心思想

### 什么是 Chain-of-Thought（CoT）

**Standard Prompt**（传统 Few-shot）：
```
Q: Roger has 5 tennis balls. He buys 2 more cans of 3 balls each. How many tennis balls does he have now?
A: 11

Q: The cafeteria had 23 apples. If they used 20 to make lunch and bought 6 more, how many do they have?
A: [模型直接输出答案]
```

**CoT Prompt**：
```
Q: Roger has 5 tennis balls. He buys 2 more cans of 3 balls each. How many tennis balls does he have now?
A: Roger started with 5 balls. 2 cans of 3 balls is 6 balls. 5 + 6 = 11. The answer is 11.

Q: The cafeteria had 23 apples. If they used 20 to make lunch and bought 6 more, how many do they have?
A: [模型输出推理步骤 + 答案]
```

区别只是：**在 Few-shot 的示例答案里，加上了推理过程**。

### 为什么有效

论文揭示了几个关键机制：

1. **分解复杂问题**：把多步骤问题拆成单步骤序列，每步都是模型"擅长"的操作
2. **可解释性**：思维链暴露了推理路径，模型不必"一步跨越"到答案
3. **错误可纠正**：如果某一步推理错了，后续步骤的 token 上下文会帮助发现矛盾
4. **涌现行为**：CoT 在小模型（<10B）上几乎没有效果，**只在足够大的模型上涌现**——这说明不是简单的 few-shot 学习，而是触发了模型的已有能力

### 规模效应（Scale）

```
模型规模        标准 Prompt    CoT Prompt
< 10B 参数       ~相同          ~相同
~100B 参数       提升          小幅提升
540B（PaLM）     明显提升       显著提升（有时翻倍）
```

这个发现对 AI 产品的启示：**Prompt 技术的效果与底层模型能力强绑定**，在小模型上跑不通的技术，在 GPT-4 / Claude 3 上可能很好用。

---

## 延伸变体

CoT 论文引发了大量跟进工作，几个重要的：

### Zero-shot CoT（Kojima et al., 2022）
不需要 Few-shot 示例，只需在 Prompt 末尾加一句：

> **"Let's think step by step."**

这句话就能激发 CoT 效果。更简单，但质量不如手写示例。

中文版等价：「请一步步思考。」

### Self-Consistency（Wang et al., 2022）
同一个问题用 CoT 生成多个答案（通过采样），取投票多数结果。

效果：在数学推理任务上，Self-Consistency 的提升比单次 CoT 更大。

产品应用：对于高价值的判断任务（分类/评估），可以跑 3-5 次取多数，代价是 token 消耗增加 3-5 倍。

### Tree of Thoughts（Yao et al., 2023）
更进一步：不是线性思维链，而是树形搜索——在每个推理节点生成多个分支，用评估函数剪枝，保留最优路径。

---

## 联系我的工程实践

### ai-prd-gen 的 Agent Prompt 设计

在 ai-prd-gen 里，每个 Agent 的 Prompt 都是隐式 CoT 设计：

```
# requirements Agent 的 Prompt（简化版）
你是一个需求分析专家。请按照以下顺序完成分析：
1. 先定义核心问题：用户真正要解决的是什么？
2. 再分析需求层次：哪些是 Must Have，哪些是 Nice to Have？
3. 最后识别关键假设：有哪些前提条件？

功能描述：{feature_idea}
目标用户：{target_user}
```

这里的"按照以下顺序"就是 CoT 的 Prompt 注入——强制模型分步骤输出，而不是直接给结论。效果：相比直接问"帮我分析这个需求"，结构化输出的完整度提升了 30-40%（主观评估）。

### Critic Agent 的 Self-Consistency 变体

在 multi-agent-report 项目里，Critic Agent 会对 Writer 的输出做评估。为了提高评估质量，我让 Critic 跑 3 次（temperature=0.7），取"三次都认为有问题"的部分才打回修改。这减少了 Critic 的误报（把好内容误判为有问题）。

---

## 局限性与边界

CoT 不是万能的：

- **事实性问题**：CoT 帮不了 LLM 知道"2024 年诺贝尔物理学奖得主是谁"——这是知识检索问题，不是推理问题
- **低资源语言**：CoT 的效果在中文上比英文弱（训练数据的推理链文本量少）
- **非语言任务**：视觉推理、纯粹的感知任务，CoT 帮助有限
- **Token 成本**：思维链 = 更多 token = 更高成本 + 更高 latency。产品上要权衡"值不值得"

---

## 一句话总结

> CoT 的本质发现：LLM 的推理能力不是"有或没有"，而是需要被**正确的 Prompt 格式激发**。"让模型把思考过程说出来"这个技巧，是 Prompt Engineering 领域迄今为止最有影响力的单一发现。

---

## 关联阅读

- **ReAct**（2023）：把 CoT 的"思考"和外部工具调用结合起来
- **Self-Consistency**（2022）：用多数投票提升 CoT 可靠性
- **Least-to-Most Prompting**（2022）：先把复杂问题分解，再逐步解决子问题

---

*[原文链接](https://arxiv.org/abs/2201.11903)*
