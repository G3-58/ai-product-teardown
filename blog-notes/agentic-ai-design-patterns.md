# 博客笔记｜Agentic AI 设计模式总结

> **来源**：综合（Andrew Ng 系列演讲 / LangGraph 官方文档 / Devin 技术分析 / OpenAI Swarm）  
> **阅读时间**：2025.01 – 2025.02  
> **一句话**：Agentic AI 的核心不是「让 LLM 自主行动」，而是「设计好的人机协作回路」。

---

## 为什么专门整理这个

做了几个 Agent 项目之后，发现很多设计决策是反复遇到的：  
「这步该不该自动化？」「Agent 挂了怎么办？」「多 Agent 怎么分工？」  
把常见模式整理出来，方便以后直接复用。

---

## 核心设计模式

### Pattern 1：Reflection（反思循环）

```
生成器 → 输出 → 评估器 → 通过? → 输出给用户
                    ↓ 不通过
                  反馈 → 生成器（重试）
```

**适用**：写作、代码生成、报告生成等有明确质量标准的任务  
**关键**：评估器的 prompt 比生成器更重要——评估标准定义了质量上限  
**陷阱**：不设 max_iter 会无限循环；评估器和生成器用同一个模型效果很差（自我审查）

**我的实践**：multi-agent-report 项目就是这个模式，Critic Agent 用低 temperature（0.2）做评估，Writer 用高 temperature（0.7）做创作，两者分开避免自我审查。

---

### Pattern 2：Tool Use + Fallback

```
LLM → 决定调用 Tool A
         ↓
    Tool A 执行
         ↓
    成功? → 继续
    失败? → 重试 / 换工具 / 降级处理 / 请求人工
```

**Fallback 层级**（从轻到重）：
1. 重试（同参数，适合网络抖动）
2. 换参数重试（适合工具参数错误）
3. 换工具（适合工具本身不可用）
4. 降级（用更简单的方法完成）
5. 人工介入（所有自动手段失败）

**陷阱**：重试不加 exponential backoff 会把第三方服务打垮；没有最终 fallback 会导致任务静默失败（用户以为成功了，其实没有）。

---

### Pattern 3：Orchestrator-Subagent 分工

```
Orchestrator（规划者）
    ├── 分解任务
    ├── 选择子 Agent
    ├── 传递上下文
    └── 汇总结果
         ↓
SubAgent A    SubAgent B    SubAgent C
（专化）      （专化）      （专化）
```

**关键设计决策**：
- Orchestrator 应该做**规划**，不应该做**执行**
- 子 Agent 之间应该**无状态**，上下文由 Orchestrator 显式传递
- 子 Agent 的粒度：太细 = overhead 大；太粗 = 重用性差

**Andrew Ng 的建议**（来自他的 Agentic AI 系列视频）：先用单 Agent + 工具解决，只有当工具太多、上下文太长时才拆分多 Agent。

---

### Pattern 4：Human-in-the-Loop 介入点设计

不是所有 Agent 决策都应该自动化。介入点的设计原则：

**必须暂停确认**：
- 不可逆操作（删除、发送、支付）
- 涉及外部系统的写操作
- 置信度 < 阈值的决策

**可以自动但要记录**：
- 可逆的读操作
- 低风险的格式转换
- 内部状态更新

**完全自动**：
- 纯计算、格式化
- 幂等操作（多次执行结果一样）

**产品视角**：介入点的设置影响用户信任建立的速度。新功能上线时多设介入点，随着用户信任度提升再逐步自动化，比一开始全自动然后出错更好。

---

### Pattern 5：State Management in Multi-Agent

多 Agent 系统里 state 管理是最容易踩坑的地方。

**LangGraph 的解法**：
- 每个 node 只能读取 state，通过返回值更新 state
- 并发写入用 reducer（`operator.add` for lists，custom reducer for complex objects）
- State 作为「真相的唯一来源」，Agent 之间不直接通信

**常见错误**：
```python
# ❌ 错误：直接修改 state
def my_node(state):
    state["results"].append(new_result)  # 并发时会丢数据
    return {}

# ✅ 正确：返回新值，让 reducer 处理
def my_node(state):
    return {"results": [new_result]}  # 配合 operator.add reducer
```

---

## Devin 的架构分析

Devin（Cognition AI，2024）是目前最接近「完整工程师 Agent」的产品，值得分析它的设计：

**已知的架构信息**：
- 有独立的代码执行沙箱（不直接在生产环境跑）
- 用 shell + browser + code editor 三类工具组合
- 有 long-term planning（生成任务计划）+ short-term execution（每步执行）的两层结构
- 遇到不确定时会主动问用户（而不是猜）

**Devin 的产品定位**：
不是「自动完成所有任务」，而是「跟人类工程师协作」——它能接手重复性高的部分，把复杂决策留给人类。这个定位比「全自动 AI 程序员」更可落地。

**我的思考**：Devin 的最大价值不是写代码，而是**上下文保持**——人类工程师换一个任务就要花时间重新理解背景，Devin 可以保持完整的项目上下文持续工作。这是 Agent 真正的差异化价值点。

---

## 2025 年值得关注的新方向

**Computer Use（计算机使用）**：  
Anthropic 的 Claude Computer Use 让 Agent 直接操控 GUI，不再依赖 API。这打开了一大批「没有开放 API 的工具」的可能性，但稳定性和安全性还有很大提升空间。

**Long-horizon Planning**：  
目前 Agent 在超过 20 步的任务上成功率急剧下降。解决方向：
- 分层规划（把 50 步计划拆成 5 个 10 步子计划）
- 更频繁的 checkpointing（每几步检查一次是否跑偏）
- 失败后的 re-planning（而不是从头再来）

**Memory 的产品化**：  
ChatGPT Memory、Claude Projects 都在解决「Agent 跨会话记住用户」的问题，但目前还比较粗糙。下一步应该是更结构化的记忆（用户偏好/项目上下文/历史决策），而不只是保存聊天记录。

---

## 一句话总结

> Agentic AI 的设计本质是在「自动化效率」和「可控性/可靠性」之间找平衡。好的 Agent 系统不追求最大自主性，而是在正确的节点让人介入、在正确的步骤做 fallback。
