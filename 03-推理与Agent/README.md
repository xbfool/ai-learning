# 03 - 推理与 Agent

> 推理模型和 Agent 系统是 2025-2026 最热门的应用方向。本章覆盖推理能力训练、DeepResearch 和 Agent 架构。

## 总览

```
推理与 Agent
├── 推理模型        → 模型 "学会思考" (CoT RL)
│   ├── OpenAI o1/o3/o4
│   ├── DeepSeek-R1
│   ├── Kimi K2.5
│   └── Step 3.5 Flash
│
├── Deep Research   → Agent "学会搜索和研究"
│   ├── Google Deep Research
│   ├── OpenAI Deep Research
│   ├── Qwen/Tongyi DeepResearch
│   └── Step DeepResearch
│
└── Agent 系统      → 模型 "学会行动"
    ├── 工具调用 (Tool Use / Function Calling)
    ├── 多 Agent 协作 (Multi-Agent)
    └── Agent Swarm (Kimi K2.5 PARL)
```

## 子文档

- [推理模型](./推理模型.md) - 从 o1 到 R1，推理能力如何训练
- [DeepResearch](./DeepResearch.md) - Qwen DeepResearch 与 IterResearch 详解
- [Agent系统](./Agent系统.md) - Agent 架构、训练与评估

## 核心脉络

### 推理训练的里程碑

```
2024.09  OpenAI o1 发布 → 证明 "思考更久可以做得更好"
2024.12  DeepSeek-R1 发布 → 开源证明 RL 能训练推理能力
2025.02  OpenAI o3 发布 → 推理能力进一步提升
2025 下半年  Kimi K2.5, Step 3.5 Flash → 推理 + Agent 融合
2026.01  o4, Qwen3.5 → 推理成为所有前沿模型标配
```

### 从推理到 Agent 的演进

```
阶段 1：单轮推理 (2024)
  模型在回答前进行长时间 "思考" (内部 CoT)
  → 数学、编程成绩大幅提升

阶段 2：工具增强推理 (2025)
  思考过程中调用工具（搜索、代码执行、计算器）
  → Deep Research 类产品
  → Kimi K2.5 工具增强后 HLE 从 40% → 50.2%

阶段 3：多 Agent 协同 (2025-2026)
  一个 "编排者" Agent 指挥多个 "执行者" Agent
  → Kimi K2.5 的 Agent Swarm (PARL)
  → 最多 100 个并行子 Agent，1500 个协调步骤
```
