# 02 - 后训练与对齐

> 预训练产出一个通用的 "基座模型"，后训练将其转化为有用的 "对话/推理/Agent 模型"。

## 总览

后训练 (Post-Training) 是将预训练模型变得有用、安全、可控的关键阶段：

```
后训练
├── SFT (监督微调)        → 教模型遵循指令格式
├── RL (强化学习)          → 优化模型行为质量
│   ├── 经典 RLHF/PPO    → 传统方案
│   ├── DPO 系列          → 简化方案
│   └── GRPO             → 当前主流方案 (DeepSeek-R1)
├── 推理 RL               → 训练模型 "学会思考"
└── Agent RL              → 训练模型 "学会行动"
```

## 后训练流水线演进

### 第一代：RLHF (2022-2023)
```
SFT → 训练奖励模型 (RM) → PPO 训练
- 需要 4 个模型：Actor, Critic, Reference, Reward Model
- 工程复杂度极高
- 代表：InstructGPT, Claude 1
```

### 第二代：DPO 简化 (2023-2024)
```
SFT → DPO (直接从偏好数据优化)
- 不需要单独的奖励模型
- 数学上等价于隐式奖励建模
- 问题：依赖静态偏好数据，缺少在线探索
- 代表：Zephyr, Llama 3 (DPO 阶段)
```

### 第三代：GRPO + 推理 RL (2025-)
```
(可选冷启动 SFT) → GRPO/REINFORCE → (拒绝采样) → 最终 SFT
- 不需要 Critic 模型
- 组内相对奖励替代 Value Function
- 支持规则/可验证奖励
- 代表：DeepSeek-R1, Kimi K2.5, Step 3.5 Flash
```

## 子文档

- [SFT 最佳实践](./SFT最佳实践.md)
- [RL 算法详解](./RL算法详解.md) - GRPO、DPO、PPO 等对比
- [RL 训练框架](./RL训练框架.md) - veRL、ROLL、AReaL 三大开源框架

## 关键洞察

1. **GRPO 是当前最实用的 RL 算法**：去掉 Critic 让工程复杂度降低一个量级
2. **可验证奖励 > 学习型奖励模型**：数学、代码等领域用规则验证效果更好
3. **Online RL > Offline 方法**：在线生成新数据的 RL (GRPO/PPO) 效果优于静态偏好数据 (DPO)
4. **冷启动 SFT 仍然重要**：DeepSeek-R1 证明少量高质量 CoT SFT 数据 + RL 效果最佳
5. **蒸馏是放大器**：大模型的推理能力可以蒸馏到 1.5B-70B 的小模型
