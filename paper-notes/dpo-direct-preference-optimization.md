# 论文笔记｜DPO: Direct Preference Optimization

> **论文**：Direct Preference Optimization: Your Language Model is Secretly a Reward Model  
> **作者**：Rafailov et al.（Stanford）  
> **发表**：NeurIPS 2023  
> **阅读时间**：2025.04  
> **关联项目**：大模型偏好对齐实验（Qwen2-7B + LoRA + DPO）

---

## TL;DR

RLHF 要训两个模型（reward model + policy），成本高、不稳定。DPO 的核心发现是：**最优策略可以从偏好数据中直接推导出闭式解**，完全绕过 reward model，把对齐问题变成一个简单的分类损失。

---

## 核心思想

### RLHF 的问题

传统 RLHF 三步走：
1. 用偏好数据训 reward model `r(x, y)`
2. 用 PPO 让 policy 最大化 `r(x, y)` 同时 KL 散度别跑太远
3. 两个模型都在显存里，训练不稳定

### DPO 的推导思路

KL 约束的 RL 目标有解析解：

```
π*(y|x) ∝ π_ref(y|x) · exp(r(x,y) / β)
```

从这个式子反推，reward function 可以用 policy 本身来表示：

```
r(x,y) = β · log[π*(y|x) / π_ref(y|x)] + β · log Z(x)
```

代入 Bradley-Terry 偏好模型，partition function `Z(x)` 消掉了，得到 DPO loss：

```
L_DPO = -E[ log σ( β·log(π_θ(y_w|x)/π_ref(y_w|x)) - β·log(π_θ(y_l|x)/π_ref(y_l|x)) ) ]
```

本质是：让模型对 chosen response 的概率提升幅度 > rejected response，且用 reference model 做归一化防止 collapse。

---

## 我的实践记录

在 Qwen2-7B 上跑 LoRA + DPO 时遇到的问题：

**问题 1：chosen/rejected 差距太小时 loss 震荡**  
→ 数据质量比数据量更关键。8k 条里我手动过滤了约 15% 差距不明显的 pair，训练稳定很多。

**问题 2：β 怎么选**  
→ β 控制与 reference model 的偏离程度，太小 = 不收敛，太大 = KL 惩罚过强变回 SFT。我最后用 β=0.1，在我的任务上效果最好。

**评测**：用 GPT-4 作为 Judge 做双盲评测，relative win-rate vs SFT baseline 提升 12%。LLM-as-a-Judge 的 prompt 设计很重要，要明确告诉模型评判维度，否则它会偏向更长的回答。

---

## 延伸问题

- DPO 对数据分布很敏感，如果 chosen/rejected 来自不同分布（一个是人写的、一个是模型生成的）效果会变差——后来 IPO / SimPO 都在解决这个问题
- 有没有办法做 online DPO（边生成边收集偏好）？Self-Play Fine-Tuning (SPIN) 在尝试这个方向

---

## 一句话总结

> DPO 是 RLHF 的「蒸馏版」——用数学推导把两个模型的训练合并成一个 BCE loss，工程上简单很多，效果在大多数场景不差。

---

*[原文链接](https://arxiv.org/abs/2305.18290)*
