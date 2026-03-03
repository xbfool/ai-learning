# Agent 系统

> LLM Agent 是将大模型从 "对话工具" 变为 "自主行动者" 的关键技术方向。

## 一、Agent 架构全景

```
Agent 架构
├── 感知 (Perception)    → 理解用户意图和环境状态
├── 推理 (Reasoning)     → 规划行动序列
├── 行动 (Action)        → 工具调用、代码执行
├── 记忆 (Memory)        → 短期/长期/检索记忆
└── 反思 (Reflection)    → 自我评估和纠错
```

## 二、Agent 范式演进

### ReAct (2023)
```
基础范式：推理 + 行动交替

Thought: 我需要搜索 X
Action: search("X")
Observation: 搜索结果...
Thought: 根据结果，我应该...
Action: ...
```
- 最简单最实用的 Agent 框架
- 仍被广泛使用

### Plan-then-Execute (2023-2024)
```
先规划，再执行：

1. 接收任务
2. 制定计划 (分解为步骤)
3. 逐步执行
4. 遇到问题时修改计划
```
- 适合复杂任务
- 代表：HuggingGPT, AutoGPT

### Reflection / Self-Critique (2024)
```
执行后反思：

1. 执行任务
2. 评估结果
3. 如果不满意 → 反思原因 → 重做
```
- 代表：Reflexion, CRITIC
- 显著提高 Agent 可靠性

### Multi-Agent (2024-2025)
```
多个专化 Agent 协作：

Orchestrator Agent
  ├── Researcher Agent    → 负责搜索
  ├── Coder Agent         → 负责编程
  ├── Reviewer Agent      → 负责审查
  └── Writer Agent        → 负责撰写
```
- 代表：AutoGen (Microsoft), CrewAI, MetaGPT
- 各 Agent 可以有不同的系统提示和工具

### Agent Swarm (2025-2026)
```
大规模并行 Agent：

Kimi K2.5 的 PARL (Parallel-Agent RL)：
  1. 编排者 Agent 接收任务
  2. 分解为可并行的子任务
  3. 动态实例化最多 100 个子 Agent
  4. 子 Agent 并行执行，最多 1500 个协调步骤
  5. 编排者汇总结果

效果：端到端延迟降低 80%，关键路径步骤减少 3-4.5x
```

## 三、工具调用 (Tool Use)

### 现状
- 工具调用已成为 LLM 的**一级训练目标**
- 不再只是 "提示工程"，而是模型原生能力
- 主要形式：Function Calling (JSON 格式)

### MCP (Model Context Protocol)
- Anthropic 提出的标准化工具调用协议
- 统一 Agent 与外部工具/数据源的交互方式
- 被越来越多的框架和产品采用

### 工具调用训练
1. **SFT 阶段**：用工具调用轨迹做监督学习
2. **RL 阶段**：用任务完成度作为奖励，训练最优工具选择
3. **关键挑战**：何时调用工具、选择哪个工具、如何解释工具结果

## 四、Agent 训练方法

### 传统方法：提示工程 (Prompt Engineering)
- 通过精心设计的系统提示定义 Agent 行为
- 优点：无需训练，快速迭代
- 缺点：可靠性有限，复杂行为难以仅通过提示控制

### 前沿方法：端到端 RL 训练 ★★★

```
核心思路：
  将 Agent 放入环境中 → 通过 RL 学习最优行为

环境 (Environment)：
  - Web 浏览 (WebArena)
  - 代码编写 (SWE-bench)
  - 搜索和研究 (GAIA, BrowseComp)
  - 电商操作、GUI 交互...

奖励 (Reward)：
  - 任务完成度 (binary: 成功/失败)
  - 效率 (步骤数)
  - 输出质量 (准确性)

算法：
  - GRPO (最常用)
  - PPO
  - REINFORCE 变体

关键论文：
  - Agent-R1: End-to-End RL for LLM Agents
  - AgentRL: Scaling Agentic Reinforcement Learning
```

### Tongyi DeepResearch 的 Agent 训练
- Agentic CPT → Agentic SFT → On-Policy RL (GRPO)
- 用 IterResearch 将问题建模为 MDP
- 端到端学习搜索策略

### Kimi K2.5 的 PARL
- Parallel-Agent Reinforcement Learning
- 训练一个可学习的编排者 Agent
- 编排者学习如何分解任务和协调子 Agent
- 子 Agent 在训练中是冻结的

## 五、评估基准

| 基准 | 评估内容 | 代表性 |
|------|---------|--------|
| GAIA | 通用 Agent 助手能力 | 多工具、多步骤 |
| WebArena | Web 浏览 Agent | GUI 交互 |
| SWE-bench | 代码修复 Agent | 真实 GitHub Issue |
| BrowseComp | 搜索能力 | Deep Research |
| FRAMES | 多文档推理 | 信息综合 |
| HLE (Humanity's Last Exam) | 极难问题 | 上限测试 |
| ARC-AGI | 抽象推理 | 泛化能力 |

## 六、Agent 记忆系统

### 短期记忆
- 上下文窗口内容（128K-1M+ tokens）
- 限制：窗口大小有限，长对话信息流失

### 长期记忆
- 向量数据库存储历史交互
- 结构化知识图谱
- 检索增强记忆

### 工作记忆
- Agent 当前任务的状态信息
- IterResearch 的 "累积知识库"

## 七、关键综述

- [A Survey on Large Language Model based Autonomous Agents](https://arxiv.org/abs/2308.11432) (Wang et al.)
- [Tool Learning with Large Language Models: A Survey](https://arxiv.org/abs/2405.17935)
- [Awesome-RL-for-LRMs](https://github.com/TsinghuaC3I/Awesome-RL-for-LRMs) - RL for Large Reasoning Models
- [AgentsMeetRL](https://github.com/thinkwee/AgentsMeetRL) - Agentic RL Awesome List
