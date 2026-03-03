# RL 算法详解

> 强化学习 (RL) 是当前大模型后训练的核心手段。本文详细对比各算法原理与适用场景。

## 一、算法全景

```
RL for LLM
├── 在线 RL (Online RL) — 训练中生成新数据
│   ├── PPO          → 经典 RLHF，需要 Critic
│   ├── GRPO         → 当前主流，无需 Critic ★★★
│   ├── REINFORCE    → 最简单的策略梯度
│   ├── RLOO         → Leave-One-Out 方差缩减
│   ├── ReMax        → REINFORCE + 基线
│   └── DAPO/DrGRPO  → GRPO 变体
│
├── 离线方法 (Offline) — 使用固定偏好数据
│   ├── DPO          → 直接偏好优化
│   ├── KTO          → 二元反馈优化
│   ├── SimPO        → 简化 DPO
│   └── Online DPO   → DPO 的在线变体
│
└── 自训练 (Self-Training)
    ├── STaR          → 自我教导推理
    └── ReST          → 自我训练 via 过滤
```

## 二、核心算法详解

### GRPO (Group Relative Policy Optimization) ★★★

**来源**：DeepSeek-R1 (2025 年 1 月)
**论文**：[arXiv:2501.12948](https://arxiv.org/abs/2501.12948)

**核心思想**：
```
对于每个 prompt x：
  1. 用当前策略采样 G 个响应：y_1, y_2, ..., y_G
  2. 对每个响应计算奖励：r_1, r_2, ..., r_G
  3. 计算组内相对优势：
     A_i = (r_i - mean(r)) / std(r)
  4. 用 PPO-clip 目标优化策略，但用组内相对优势替代 Value Function
```

**为什么 GRPO 成为主流**：
- **去掉 Critic 模型**：PPO 需要一个与 Actor 一样大的 Critic 模型 → GRPO 直接用组内比较替代
- **GPU 显存节省 ~50%**：不需要 Critic 的参数和优化器状态
- **实现更简单**：Critic 的训练本身就是个难题
- **效果不差**：在数学、代码等可验证领域，GRPO 效果与 PPO 相当甚至更好

**关键超参数**：
- G (组大小)：通常 8-64，更大的 G 提供更稳定的优势估计
- KL 惩罚系数：控制策略偏离参考策略的程度
- Clip 范围：PPO-clip 的裁剪阈值

### PPO (Proximal Policy Optimization)

**经典 RLHF 算法**
```
需要 4 个模型：
  Actor (策略模型) → 被训练
  Critic (价值模型) → 估计 V(s)
  Reference (参考模型) → KL 正则化
  Reward Model → 提供奖励信号

训练循环：
  1. Actor 生成响应
  2. Reward Model 打分
  3. Critic 估计优势 A = R - V(s)
  4. Actor 用 PPO-clip 目标优化
  5. Critic 用 Value Loss 优化
```

**优势**：理论基础扎实，适用范围广
**劣势**：4 个模型内存开销大，Critic 训练不稳定，工程复杂

### DPO (Direct Preference Optimization)

**论文**：[arXiv:2305.18290](https://arxiv.org/abs/2305.18290)

**核心思想**：
```
给定偏好对 (x, y_win, y_lose):
  Loss = -log σ(β · (log π(y_win|x)/π_ref(y_win|x) - log π(y_lose|x)/π_ref(y_lose|x)))

等价于：隐式定义了一个奖励函数 r(x,y) = β · log(π(y|x)/π_ref(y|x))
```

**优势**：
- 不需要单独的奖励模型
- 训练稳定，实现简单
- 只需偏好数据对

**劣势**：
- 离线方法，受限于固定偏好数据
- 缺少在线探索能力
- 效果通常不如在线 RL 方法

### DAPO (Dynamic sampling Policy Optimization)

- 基于 GRPO 的改进变体
- 在 Qwen2.5-32B 上用 veRL 训练
- AIME 2024 达到 50 分，超越 DeepSeek 的 GRPO
- **开源 SOTA RL 算法之一**

## 三、奖励信号设计

### 类型

| 奖励类型 | 描述 | 适用场景 | 代表 |
|---------|------|---------|------|
| 规则/可验证奖励 | 程序化验证答案正确性 | 数学、代码 | DeepSeek-R1 |
| 学习型奖励模型 | 训练一个模型给回答打分 | 通用对话 | InstructGPT |
| AI 反馈 (RLAIF) | 用另一个 LLM 做裁判 | 开放域 | Constitutional AI |
| 过程奖励 (PRM) | 奖励每个推理步骤 | 数学推理 | "Let's Verify Step by Step" |
| 结果奖励 (ORM) | 只奖励最终结果 | 通用 | 大多数 RLHF |

### DeepSeek-R1 的奖励设计
```
数学题：答案正确 → r = 1，错误 → r = 0
代码题：通过所有测试用例 → r = 1，否则 → r = 0
格式奖励：输出格式符合要求 → 额外 r
```
- **简单、可靠、可扩展**
- 不需要训练奖励模型
- 但仅适用于有客观答案的领域

## 四、在线 vs 离线 RL 的权衡

| 维度 | 在线 RL (GRPO/PPO) | 离线 (DPO) |
|------|-------------------|------------|
| 数据 | 训练中在线生成 | 固定偏好数据集 |
| 探索 | 有（生成多样响应） | 无 |
| 效果 | 更强 | 较弱 |
| 计算成本 | 高（需要 rollout） | 低 |
| 工程复杂度 | 高 | 低 |
| 适用场景 | 追求极致效果 | 快速迭代/资源受限 |

**共识 (2025)**：在线 RL 效果明显优于离线方法。对于追求最佳性能的场景，GRPO 是首选。

## 五、RL 训练的涌现现象

### DeepSeek-R1-Zero 的发现
- 纯 RL 训练（无 SFT）也能涌现 Chain-of-Thought 推理
- 涌现行为包括：
  - **自我验证** ("let me check...")
  - **反思** ("wait, that's wrong...")
  - **探索多条路径** ("alternatively...")
  - **"Aha" 时刻**：突然发现新的推理策略

### 启示
- RL 不仅仅是 "对齐"，它能**发现新的能力**
- 推理能力可能是通过 RL 自然涌现的，而非需要显式教导
- 这从根本上改变了对 RL 在 LLM 训练中角色的理解
