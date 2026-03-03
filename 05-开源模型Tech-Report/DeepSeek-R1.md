# DeepSeek-R1 技术报告解读

> DeepSeek-R1 证明了纯强化学习可以让 LLM 涌现复杂推理行为，以约 1/10 的成本匹配 OpenAI o1，是 2025 年最具影响力的开源模型之一。

- **论文**: [DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning](https://arxiv.org/abs/2501.12948)
- **发布**: 2025 年 1 月 20 日
- **基座模型**: DeepSeek-V3-Base
- **GitHub**: [deepseek-ai/DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1)
- **HuggingFace**: [deepseek-ai/DeepSeek-R1](https://huggingface.co/deepseek-ai/DeepSeek-R1)
- **Nature 发表**: [nature.com](https://www.nature.com/articles/s41586-025-09422-z)

---

## 一、核心贡献

1. **R1-Zero 实验**：证明纯 RL（无 SFT）可以让 LLM 涌现推理行为
2. **四阶段训练流水线**：冷启动 SFT → RL → 拒绝采样 SFT → 二次 RL
3. **GRPO 算法的核心应用**：无需 Critic 模型的高效 RL
4. **蒸馏**：将推理能力蒸馏到 1.5B-70B 小模型
5. **开源**：完整模型权重和蒸馏模型

---

## 二、DeepSeek-R1-Zero：纯 RL 实验 ★★★

> 里程碑式实验：直接从基座模型出发，不用任何 SFT 数据，纯 RL 训练涌现推理能力。

### 训练方式

- 从 DeepSeek-V3-Base 出发
- **完全不使用 SFT 数据**
- 仅使用 GRPO 算法进行 RL 训练
- 奖励信号仅基于最终答案正确性（与标准答案对比）
- 不对推理过程施加任何约束

### 涌现行为 (Emergent Behaviors)

随着训练推进，模型自发涌现出：

```
1. 延长思维链：对困难问题自动增加推理步骤
   训练初期：简短回答
   训练后期：几千 Token 的详细推导

2. 自我反思：回顾之前的步骤，发现错误
   "Wait, let me re-examine step 3..."

3. 自我验证：检查自身推理的正确性
   "Let me verify this by substituting back..."

4. 动态策略调整：当早期方法失败时切换策略
   "This approach doesn't work, let me try a different method..."
```

### "Aha Moment" (顿悟时刻)

DeepSeek 团队报告的标志性现象：

```
模型在推理链中突然停下来：
"Hmm, wait, I think I made an error in the above steps.
 Let me reconsider..."

然后自我纠正并得到正确答案。
这种行为完全通过 RL 信号自发涌现，未经任何人为设计。
```

**学术讨论**：Sea AI Lab 等后续研究对"aha moment"是否真正是 RL 中涌现的现象提出了质疑，认为自我反思模式可能在基座模型中已存在。这仍是活跃的研究话题。

### R1-Zero 的局限

| 问题 | 表现 |
|------|------|
| 可读性差 | 逻辑正确但结构混乱 |
| 语言混杂 | 推理过程中混合中英文等多种语言 |
| 无限重复 | 有时陷入循环推理 |
| 缺乏格式 | 没有 Markdown 等格式标记 |

这些局限直接推动了完整 DeepSeek-R1 的开发。

---

## 三、GRPO 算法详解

> Group Relative Policy Optimization，最初在 DeepSeekMath 论文 ([arXiv:2402.03300](https://arxiv.org/abs/2402.03300)) 中提出。

### 与 PPO 的关键区别

```
PPO 需要 4 个模型：
  Actor (策略模型) + Critic (价值函数) + Reference (参考模型) + Reward Model
  Critic 模型通常与 Actor 同等规模 → 巨大的显存和计算开销

GRPO 只需要 2 个模型：
  Actor + Reference
  完全去掉 Critic → 显存和计算减半
```

### 核心机制：基于组的相对优势估计

```
GRPO 流程：

对每个问题 q：
  1. 从当前策略 π_θ 采样一组 (group) 输出: {o_1, o_2, ..., o_G}
  2. 计算每个输出的奖励: r_1, r_2, ..., r_G
  3. 计算组统计: μ = mean(r), σ = std(r)
  4. 每个输出的优势: A_i = (r_i - μ) / σ
  5. 用优势更新策略（带 KL 约束）

直觉：
  不需要绝对的"好坏"判断
  只需要在一组回答中区分"相对更好"和"相对更差"
```

### R1 中的 GRPO 超参数

| 参数 | 值 |
|------|-----|
| 学习率 | 3e-6 |
| KL 系数 | 0.001 |
| GRPO clip ratio ε | 10 |
| 采样温度 | 1 (rollout) |
| 每题采样数 | 16 |
| 最大长度 | 32,768 tokens |
| 每步唯一问题数 | 32 |
| 训练批大小 | 512/步 |

---

## 四、DeepSeek-R1 四阶段训练流水线 ★★★

```
┌───────────────────────────────────────────────────────┐
│              DeepSeek-R1 训练流水线                      │
│                                                        │
│  V3-Base → [冷启动SFT] → [推理RL] → [拒绝采样SFT] → [二次RL] → R1 │
│              阶段1         阶段2        阶段3          阶段4      │
└───────────────────────────────────────────────────────┘
```

### 阶段 1：冷启动 (Cold Start)

| 项目 | 详情 |
|------|------|
| **目标** | 避免从 base 模型直接 RL 的不稳定性 |
| **数据量** | 数千条长思维链 (Long CoT) 数据 |
| **数据来源** | few-shot 提示生成 + R1-Zero 输出筛选 + 人工标注后处理 |
| **方法** | 对 V3-Base 做 SFT |

### 阶段 2：面向推理的 RL (Reasoning-Oriented RL)

| 项目 | 详情 |
|------|------|
| **目标** | 大规模训练推理能力 |
| **任务** | 数学、代码、科学等推理密集型任务 |
| **算法** | GRPO |
| **奖励** | 规则/可验证奖励（数学答案验证、代码执行） |
| **创新** | 引入**语言一致性奖励**：计算 CoT 中目标语言词汇比例，缓解语言混杂 |
| **训练** | 至接近收敛 |

### 阶段 3：拒绝采样 + SFT (Rejection Sampling + SFT)

| 项目 | 详情 |
|------|------|
| **目标** | 拓展非推理任务能力 (写作、QA、自我认知) |
| **方法** | 在 RL 检查点上拒绝采样生成新 SFT 数据 |
| **数据** | RL 生成的高质量推理数据 + V3 的写作/事实/对话数据 |
| **输出** | 全面能力的 SFT 模型 |

### 阶段 4：二次 RL (Secondary RL)

| 项目 | 详情 |
|------|------|
| **目标** | 最终优化有用性 + 无害性 + 推理 |
| **方法** | 对 SFT 检查点再次 RL |
| **重点** | 提升 helpfulness、增强 harmlessness |
| **输出** | 最终 DeepSeek-R1 模型 |

---

## 五、蒸馏模型 (Distilled Models)

> 将 R1 的推理能力蒸馏到 6 个小模型，其中 32B 版本超越 OpenAI o1-mini。

### 蒸馏方法

- 使用 DeepSeek-R1 生成约 **800K 条**推理样本
- 直接用这些样本对小模型做 SFT（不是 KD，而是纯 SFT on R1 outputs）

### 蒸馏模型性能

| 蒸馏模型 | 基座 | AIME 2024 | MATH-500 | GPQA Diamond | LiveCodeBench |
|----------|------|-----------|----------|--------------|---------------|
| R1-Distill-Qwen-1.5B | Qwen-2.5-1.5B | - | 83.9% | - | 16.9% |
| R1-Distill-Qwen-7B | Qwen-2.5-7B | - | - | - | - |
| R1-Distill-Llama-8B | Llama-3.1-8B | - | 89.1% | 49.0% | 39.6% |
| R1-Distill-Qwen-14B | Qwen-2.5-14B | - | - | - | - |
| **R1-Distill-Qwen-32B** | Qwen-2.5-32B | **72.6%** | **94.3%** | **62.1%** | **57.2%** |
| R1-Distill-Llama-70B | Llama-3.3-70B | - | - | - | - |

**关键成果**: R1-Distill-Qwen-32B 在多个基准上**超越 OpenAI o1-mini**，为密集模型树立新 SOTA。

---

## 六、DeepSeek-R1 性能基准

### R1 vs OpenAI o1-1217

| 基准 | DeepSeek-R1 | OpenAI o1-1217 | 胜者 |
|------|------------|----------------|------|
| AIME 2024 (pass@1) | **79.8%** | 79.2% | R1 |
| MATH-500 | **97.3%** | 96.4% | R1 |
| Codeforces (Elo) | 2029 | - | - |
| Codeforces (百分位) | 96.3% | 96.6% | o1 (微弱) |
| GPQA Diamond | 71.5% | **75.7%** | o1 |
| LiveCodeBench | 65.9% | - | - |

### 总体评价

```
数学推理：R1 略微领先 o1
编程竞赛：两者非常接近
通用知识 (GPQA)：o1 更优

成本对比：R1 的 API 价格比 o1 低 90-95%
性能/成本比：R1 具有压倒性优势
```

---

## 七、DeepSeek-V3 与 R1 的关系

```
DeepSeek-V3-Base (基座模型)
  │
  ├── DeepSeek-V3 (Chat 版本)
  │     经过标准后训练 (SFT + DPO)
  │     → 通用对话能力
  │
  └── DeepSeek-R1 (推理版本)
        经过四阶段流水线 (SFT + RL + 拒绝采样 + RL)
        → 强大推理能力

V3 提供高效基础设施 (MoE + MLA)
R1 在此基础上叠加推理增强层
```

---

## 八、影响与后续

### 对行业的影响

1. **证明了开源 RL 训练推理能力的可行性**：打破 OpenAI o1 的闭源垄断
2. **GRPO 成为主流算法**：Kimi K2.5、Step 3.5 Flash、GLM-5 等都采用 GRPO 或其变体
3. **蒸馏范式**：大量后续工作使用 R1 输出做蒸馏 (Open-R1, Sky-T1 等)
4. **成本标杆**：证明前沿推理能力不需要天文数字的训练预算

### 后续模型

- **DeepSeek-R1-0528** (2025.05): 改进版本
- 社区蒸馏：大量基于 R1 输出训练的开源模型

---

## 九、核心参考

| 资源 | 链接 |
|------|------|
| DeepSeek-R1 论文 | [arXiv:2501.12948](https://arxiv.org/abs/2501.12948) |
| DeepSeek-R1 GitHub | [GitHub](https://github.com/deepseek-ai/DeepSeek-R1) |
| DeepSeek-R1 HuggingFace | [HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1) |
| GRPO 深度解析 | [Cameron Wolfe](https://cameronrwolfe.substack.com/p/grpo) |
| R1 详解 (HuggingFace) | [HF Blog](https://huggingface.co/blog/NormalUhr/deepseek-r1-explained) |
| HuggingFace LLM Course R1 章节 | [HF Course](https://huggingface.co/learn/llm-course/en/chapter12/3) |
| Aha Moment 分析 (Sea AI Lab) | [Blog](https://sail.sea.com/blog/articles/62) |
| Nature 发表 | [Nature](https://www.nature.com/articles/s41586-025-09422-z) |
| Sebastian Raschka 技术巡礼 | [Substack](https://magazine.sebastianraschka.com/p/technical-deepseek) |
| DeepSeek 模型完全指南 (BentoML) | [Blog](https://www.bentoml.com/blog/the-complete-guide-to-deepseek-models-from-v3-to-r1-and-beyond) |
