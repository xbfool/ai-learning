# Deep Research

> Deep Research 是 2025 年最成功的 Agent 应用形态之一：模型自主搜索、阅读、推理，生成深度研究报告。

## 一、什么是 Deep Research？

```
用户提出一个复杂问题
  ↓
Agent 自动分解为子问题
  ↓
循环执行：搜索 → 阅读 → 推理 → 评估 → 再搜索
  ↓ (10-50+ 轮)
综合所有发现，生成结构化研究报告（含引用）
```

**核心差异**于传统 RAG：
- RAG：搜一次 → 读一次 → 回答
- Deep Research：搜 N 次 → 读 N 次 → 推理 N 次 → 报告

## 二、各家对比

| 维度 | Google | OpenAI | Qwen/Tongyi | Step |
|------|--------|--------|-------------|------|
| **模型** | Gemini 2.0+ | o3 系列 | QwQ/Qwen3-30B-A3B | Step 系列 |
| **开源** | 否 | 否 | **是** ★ | **是** ★ |
| **搜索引擎** | Google Search | Bing | 可配置 | 可配置 |
| **推理方式** | Extended Thinking | Extended Thinking | Extended Thinking | Extended Thinking |
| **RL 训练** | 未公开 | 未公开 | **GRPO** | 未详述 |
| **报告质量** | 高 | 最高 | 高 | 高 |
| **论文** | - | - | [arXiv:2510.24701](https://arxiv.org/html/2510.24701v1) | [arXiv:2512.20491](https://arxiv.org/pdf/2512.20491) |

## 三、Tongyi DeepResearch 详解

### 基本信息
- **模型**：30.5B 参数，3.3B 激活（基于 Qwen3-30B-A3B-Base）
- **论文**：[Tongyi DeepResearch Technical Report](https://arxiv.org/html/2510.24701v1)
- **开源**：是

### 训练流水线

```
Qwen3-30B-A3B-Base
  │
  ├── 1. Agentic CPT (持续预训练)
  │     用大规模合成 Agent 轨迹做持续预训练
  │     → 模型学会 "搜索-阅读-推理" 的基本模式
  │
  ├── 2. Agentic SFT
  │     用 ReAct 格式和 IterResearch 格式的轨迹做 SFT
  │     → 模型学会两种 Agent 交互范式
  │
  └── 3. On-Policy RL (GRPO)
        Token 级策略梯度
        Leave-One-Out 优势估计
        选择性负样本过滤（提高稳定性）
        → 模型学会 "什么时候搜、搜什么、什么时候停"
```

### IterResearch 框架 ★★★

**核心创新**：将 Deep Research 问题建模为**马尔科夫决策过程 (MDP)**

```
状态 (State)：当前已收集的信息 + 未回答的子问题
动作 (Action)：搜索、阅读、推理、总结、提出新问题
转移 (Transition)：执行动作后状态更新
奖励 (Reward)：最终报告的质量（准确性、完整性、引用质量）

→ 可以直接用 RL (GRPO) 端到端训练
→ 不再是 "提示工程"，而是 "端到端学习"
```

#### IterResearch 两种模式
1. **标准模式**：单轮搜索-推理循环
2. **Heavy 模式**（Test-Time Scaling）：
   - 结构化多轮合成
   - 每轮结束时重构上下文（减少噪声累积）
   - 类似 "渐进式文献综述"

### 性能基准
| 基准 | 得分 |
|------|------|
| Humanity's Last Exam | 32.9 |
| BrowseComp | 43.4 |
| BrowseComp-ZH | 46.7 |
| WebWalkerQA | 72.2 |
| GAIA | 70.9 |
| FRAMES | 90.6 |

## 四、Step DeepResearch

- **开源**：[GitHub - stepfun-ai/StepDeepResearch](https://github.com/stepfun-ai/StepDeepResearch)
- **论文**：[arXiv:2512.20491](https://arxiv.org/pdf/2512.20491)
- 基于 Step 系列模型
- 与 Tongyi DeepResearch 类似的 Agent Loop

## 五、Deep Research 的关键技术

### Agent Loop 设计
```python
# 简化的 Deep Research Loop
def deep_research(question):
    plan = decompose(question)       # 分解子问题
    knowledge_base = {}              # 累积知识库

    for sub_question in plan:
        while not sufficient(knowledge_base, sub_question):
            query = formulate_query(sub_question, knowledge_base)  # 生成搜索查询
            results = search(query)                                 # 搜索
            extracted = read_and_extract(results)                   # 阅读和提取
            knowledge_base.update(extracted)                        # 更新知识库
            plan = maybe_refine_plan(plan, knowledge_base)          # 可能修改计划

    report = synthesize(question, knowledge_base)  # 综合报告
    return report
```

### RL 训练的奖励信号
1. **事实准确性**：报告中的陈述是否正确
2. **完整性**：是否覆盖了问题的所有方面
3. **引用质量**：引用是否相关和准确
4. **效率**：用更少的搜索步骤获得更好的结果

### 关键挑战
- **上下文管理**：多轮搜索积累大量文本 → 需要智能压缩/总结
- **停止决策**：什么时候停止搜索？→ RL 学习最优停止策略
- **噪声累积**：搜索结果可能包含错误信息 → IterResearch Heavy 模式的上下文重构
- **评估困难**：开放域报告质量难以自动评估

## 六、实践建议

### 构建自己的 Deep Research 系统
1. **最简方案**：用强推理模型 + 搜索 API + ReAct Prompt
2. **进阶方案**：微调模型的搜索/停止行为（SFT on agent trajectories）
3. **最佳方案**：端到端 RL 训练（如 Tongyi DeepResearch）

### 开源资源
- [Qwen-Agent](https://github.com/QwenLM/Qwen-Agent) - Qwen 的 Agent 框架
- [StepDeepResearch](https://github.com/stepfun-ai/StepDeepResearch) - Step 的开源实现
- Tongyi DeepResearch 模型和代码

---

## 七、Tongyi DeepResearch 论文深度解析

> 论文：[Tongyi DeepResearch Technical Report (arXiv:2510.24701)](https://arxiv.org/abs/2510.24701)
> 团队：Tongyi DeepResearch Team (阿里巴巴)
> 模型：Tongyi-DeepResearch-30B-A3B（30.5B 总参数，3.3B 激活）

### 7.1 核心定位与贡献

Tongyi DeepResearch 是首个性能媲美 OpenAI DeepResearch 的**全开源 Web Agent**。其核心贡献包括：

1. **IterResearch 框架**：一种全新的 Agent 交互范式，将深度研究建模为马尔科夫决策过程（MDP），通过动态工作区重构解决长程任务中的"认知窒息"和"噪声污染"问题
2. **全自动数据合成流水线（AgentFounder）**：无需人工标注，从知识图谱、文档、工具使用轨迹中自动生成大规模训练数据
3. **端到端训练框架**：Agentic CPT → Agentic SFT → On-Policy RL (GRPO) 三阶段渐进训练
4. **开源模型与代码**：完整开放模型权重、推理代码和训练方法论

### 7.2 IterResearch 框架：MDP 建模详解

#### 7.2.1 问题：单上下文范式的困境

传统 ReAct 范式将所有历史信息累积在一个不断扩展的上下文中，这带来两个致命问题：

- **认知工作区窒息（Cognitive Workspace Suffocation）**：随着上下文不断膨胀，模型的有效推理窗口被海量历史信息挤占，推理质量急剧下降
- **不可逆噪声污染（Irreversible Noise Contamination）**：搜索过程中引入的无关信息和早期错误会永久残留在上下文中，持续干扰后续决策

#### 7.2.2 解决方案：研究轮次与工作区重构

IterResearch 的核心设计是将一个复杂研究任务分解为**一系列离散的"研究轮次"（Research Rounds）**。每个轮次结束时，Agent 会**重构一个精简的工作区**，只保留上一轮最关键的输出，从而维持清晰的"认知焦点"。

```
研究轮次 1:
  工作区 = [原始问题, 空报告]
  → 思考 → 搜索 → 阅读 → 推理
  → 更新中央报告（Central Report）
  → 丢弃临时上下文

研究轮次 2:
  工作区 = [原始问题, 更新后的报告]  ← 重构！
  → 思考 → 搜索 → 阅读 → 推理
  → 再次更新中央报告
  → 丢弃临时上下文

...

研究轮次 N:
  工作区 = [原始问题, 最终报告]
  → 判断信息是否充分 → 输出最终答案
```

**关键洞察**："合成与重构"的迭代过程使得 Agent 在整个长程任务中始终保持高质量推理。

#### 7.2.3 MDP 形式化定义

IterResearch 将深度研究的执行过程形式化为一个**信息获取马尔科夫决策过程（Information Acquisition MDP）**：

| MDP 元素 | 定义 | 说明 |
|----------|------|------|
| **状态 S** | `s_t = (q, R_t, c_t)` | `q` = 原始研究问题；`R_t` = 当前中央报告（综合所有历史发现和当前研究进度）；`c_t` = 最近一次工具交互的即时上下文 |
| **动作 A** | 查询生成、信息摘要、推理、回答 | Agent 迭代执行：查询生成（信息获取）、摘要（信息压缩）、推理（推断），以发现与任务相关的原子事实和引用 |
| **转移 T** | `s_{t+1} = T(s_t, a_t)` | 在轮次间，转移函数保留更新后的报告 `R_{t+1}`，同时丢弃临时信息 `c_t`，确保马尔科夫性质 |
| **奖励 R** | 最终报告质量 | 基于最终报告的准确性、完整性、引用质量等维度评估 |

**马尔科夫性质保证**：每个状态只包含必要组成部分（问题 + 演化报告 + 即时上下文），轮次间通过状态转移函数清除临时信息，使得当前状态完整编码了所有做决策所需的信息，不依赖更早的历史。

```
MDP 状态转移图：

s_0 = (q, R_0=空, c_0=空)
  │
  ├── a_0: 生成搜索查询
  │     环境返回搜索结果
  ├── a_1: 阅读并提取关键信息
  ├── a_2: 推理、更新报告 R_1
  │
  ▼ 状态转移：丢弃 c_0，保留 R_1
s_1 = (q, R_1, c_1=新上下文)
  │
  ├── a_3: 根据 R_1 中的空白，生成新查询
  ├── a_4: 搜索、阅读、提取
  ├── a_5: 推理、更新报告 R_2
  │
  ▼ 状态转移
s_2 = (q, R_2, c_2=新上下文)
  │
  ...
  ▼
s_N: Agent 判断信息充分 → 输出最终答案
```

#### 7.2.4 IterResearch 的两种推理模式

| 模式 | 适用场景 | 特点 |
|------|---------|------|
| **标准模式（ReAct 兼容）** | 评估模型核心内在能力 | 标准 Thought-Action-Observation 循环，严格遵循 ReAct 格式 |
| **Heavy 模式（Test-Time Scaling）** | 释放模型最大性能上限 | 结构化多轮合成，每轮结束时重构上下文，类似"渐进式文献综述" |

**Heavy 模式详解**：

Heavy 模式是 IterResearch 的测试时扩展（Test-Time Scaling）策略。其核心思想是：通过增加推理时间的计算量（更多轮次、更深推理），而非增加模型参数，来提升最终输出质量。

```
Heavy 模式工作流程：

Round 1: 初始探索
  ├── 分析问题，制定研究计划
  ├── 执行初步搜索（多次）
  ├── 阅读、提取、初步推理
  └── 生成"进度报告 v1"（包含已知发现 + 待探索方向）

Round 2: 深化研究
  ├── 【重构工作区】只保留进度报告 v1 + 原始问题
  ├── 基于进度报告中的"待探索方向"，生成精准查询
  ├── 搜索 → 阅读 → 交叉验证
  └── 生成"进度报告 v2"

Round 3-N: 迭代深化
  ├── 每轮重构，每轮都在更精炼的信息基础上推理
  └── 直到覆盖所有方面或达到计算预算

最终轮: 综合输出
  └── 将最终进度报告整理为结构化研究报告
```

### 7.3 AgentFounder：全自动数据合成引擎

Tongyi DeepResearch 训练管线的一大亮点是其**全自动数据合成流水线**，完全无需人工标注。

#### 7.3.1 数据合成三步骤

```
步骤 1：问题合成（Question Synthesis）
  ├── 基于实体锚定的开放世界记忆（Entity-Anchored Open-World Memory）
  ├── 从知识图谱中选择实体节点
  ├── 通过"层级式扩展（Layer-wise Expansion）"策略生成复杂研究问题
  │     ├── 并集操作（Union）：组合多个实体关系
  │     └── 交集操作（Intersection）：要求同时满足多个条件
  └── 消除冗余和推理捷径

步骤 2：Agent 行为数据生成
  ├── 一阶动作合成（First-order Action Synthesis, FAS）
  │     ├── 规划动作（Planning Action）：使用开源模型分析和分解问题
  │     ├── 推理动作（Reasoning Action）：两阶段过程，由大模型引导生成完整推理链
  │     └── 通过拒绝采样（Rejection Sampling）保证质量
  │
  └── 高阶动作合成（Higher-order Action Synthesis, HAS）
        ├── 多决策动作合成（Multi-Decision Action Synthesis）
        ├── 将丢弃或部分完成的 Agent 轨迹重新映射为逐步决策数据集
        └── 提高样本效率和多样性

步骤 3：在训练管线中使用 Agent 数据
  ├── CPT 阶段：大规模合成轨迹（推理链、工具调用、决策树）
  ├── SFT 阶段：ReAct 格式 + IterResearch 格式的专家轨迹
  └── RL 阶段：在线策略交互数据
```

#### 7.3.2 知识图谱驱动的数据生成

AgentFounder 引擎的核心能力：

- **输入**：原始语料库（文档、网页爬取数据、知识图谱）
- **处理**：将原始语料重组为实体锚定的 QA 对
- **输出**：覆盖推理链、工具调用、决策树的多样化 Agent 轨迹

具体工具包括：
- **AgentFounder**：用于预训练阶段 —— 合成一阶和高阶动作数据
- **WebSailor-V2**：用于后训练阶段 —— 通过知识图谱上的随机游走生成 QA 数据集
- **WebShaper**：用于后训练阶段 —— 基于层级式扩展的信息检索策略，提升数据质量

### 7.4 三阶段训练管线详解

#### 7.4.1 阶段一：Agentic 持续预训练（Agentic CPT）

**目标**：在标准预训练和复杂 Agent 行为之间搭建桥梁

```
输入：Qwen3-30B-A3B-Base
训练方式：Next-Token Prediction（标准自回归损失）
数据来源：AgentFounder 生成的大规模合成 Agent 行为数据
  ├── 搜索-阅读-推理轨迹
  ├── 工具调用序列
  └── 多步决策树

两阶段上下文扩展：
  阶段 1a: 32K 上下文长度
  阶段 1b: 扩展到 128K 上下文长度

产出：模型学会"搜索-阅读-推理"的基本行为模式
```

#### 7.4.2 阶段二：Agentic 监督微调（Agentic SFT）

**目标**：冷启动，让模型学会结构化的 Agent 交互格式

```
数据格式：
  ├── ReAct 格式轨迹：Thought → Action → Observation 循环
  └── IterResearch 格式轨迹：轮次间工作区重构 + 中央报告更新

训练效果：
  ├── 模型掌握两种 Agent 交互范式
  ├── 学会规划和工具使用的 schema
  └── 建立格式一致的规划和工具使用能力
```

#### 7.4.3 阶段三：在线策略强化学习（On-Policy RL with GRPO）

**目标**：让模型通过自我进化学会"什么时候搜、搜什么、什么时候停"

这是整个训练管线中最关键的阶段，详见下文第九节的 GRPO 训练详解。

---

## 八、IterResearch vs ReAct：两种 Agent 范式深度对比

### 8.1 ReAct 范式回顾

ReAct（Reasoning + Acting）是经典的多轮推理格式，交互结构为：

```
Thought: 我需要搜索关于 X 的信息...
Action: search("X 相关查询")
Observation: [搜索结果]
Thought: 根据搜索结果，我发现 Y，但还需要了解 Z...
Action: search("Z 相关查询")
Observation: [搜索结果]
...
Thought: 现在我有足够信息了
Action: finish(最终答案)
```

**核心特征**：所有历史 Thought/Action/Observation 保留在同一个上下文中，上下文线性增长。

### 8.2 IterResearch 范式

```
=== 研究轮次 1 ===
工作区: [问题 Q, 报告 R_0=空]

Thought: 分析问题，制定搜索策略...
Action: search("查询 1")
Observation: [结果 1]
Thought: 提取关键信息，整合到报告中...
Action: search("查询 2")
Observation: [结果 2]
→ 更新报告 R_1

=== 轮次切换：工作区重构 ===
  丢弃轮次 1 的所有 Observation 和临时 Thought
  只保留 R_1

=== 研究轮次 2 ===
工作区: [问题 Q, 报告 R_1]  ← 精简后的工作区

Thought: 根据报告 R_1，还缺少关于 Z 的信息...
Action: search("针对 Z 的精准查询")
Observation: [结果]
→ 更新报告 R_2

=== 轮次切换：工作区重构 ===
...
```

### 8.3 核心差异对比

| 维度 | ReAct | IterResearch |
|------|-------|-------------|
| **上下文管理** | 全部历史累积在单一上下文中 | 每轮重构，只保留中央报告 |
| **上下文增长** | 线性增长，O(T) | 受控增长，报告大小有上限 |
| **推理质量** | 随轮次增加而下降（上下文污染） | 保持稳定（每轮都在干净工作区推理） |
| **信息持久性** | 所有信息永久保留（包括噪声） | 只有经过推理筛选的信息进入报告 |
| **错误传播** | 早期错误永久污染后续决策 | 每轮重构时可纠正早期错误 |
| **长程任务** | 性能急剧下降（10+ 轮后） | 可稳定执行 50+ 轮 |
| **MDP 建模** | 状态包含全部历史（非马尔科夫） | 天然满足马尔科夫性质 |
| **RL 训练** | 难以直接用 RL 优化 | MDP 结构便于端到端 RL 训练 |
| **计算效率** | 每步需处理全部历史上下文 | 每步只处理精简工作区 |

### 8.4 为什么 IterResearch 更适合 RL 训练？

```
ReAct 的问题：
  状态 = 全部历史（Thought_1, Action_1, Obs_1, ..., Thought_t, Action_t, Obs_t）
  → 状态空间爆炸
  → 信用分配困难（哪个早期动作导致了最终成功/失败？）
  → 价值函数估计困难

IterResearch 的优势：
  状态 = (问题, 当前报告, 即时上下文)  ← 紧凑、结构化
  → 状态空间可控
  → 信用分配更清晰（每轮的贡献体现在报告的增量更新中）
  → 天然的马尔科夫性质，适合标准 RL 算法
```

### 8.5 实际性能对比

在 Tongyi DeepResearch 的实验中，两种范式的表现：

- **ReAct 模式**：用于严格评估模型核心内在能力，不使用任何提示工程，模型能自主遵循 Thought-Action-Observation 循环执行多轮迭代
- **IterResearch Heavy 模式**：通过测试时扩展策略释放模型最大性能上限，在 BrowseComp、Humanity's Last Exam 等基准上取得更高分数

**关键结论**：同一个模型在两种范式下都能正常工作（因为 SFT 阶段同时训练了两种格式），但 IterResearch Heavy 模式通过更好的上下文管理和测试时扩展，能发挥出模型的最大潜力。

---

## 九、GRPO 训练 Agent 行为：技术深度解析

### 9.1 GRPO 基础原理

GRPO（Group Relative Policy Optimization）最初由 DeepSeek 在 [DeepSeekMath](https://arxiv.org/abs/2402.03300) 中提出，是 PPO 的一种变体，核心创新在于**用组内相对比较替代价值网络**来估计优势函数。

#### 标准 GRPO 流程

```
对于每个问题 q：
  1. 从当前策略 π_θ 中采样 G 个输出 {o_1, o_2, ..., o_G}
  2. 对每个输出计算奖励 {r_1, r_2, ..., r_G}
  3. 组内归一化计算优势：
     Â_i = (r_i - mean(r)) / std(r)
  4. 用优势加权的策略梯度更新参数
```

#### GRPO 损失函数

```
L_GRPO(θ) = -E_q [
    (1/G) Σ_{i=1}^{G} (1/|o_i|) Σ_{t=1}^{|o_i|} (
        min(
            ρ_{i,t} · Â_i,
            clip(ρ_{i,t}, 1-ε, 1+ε) · Â_i
        )
        - β · D_KL(π_θ || π_ref)
    )
]

其中：
  ρ_{i,t} = π_θ(o_{i,t} | q, o_{i,<t}) / π_old(o_{i,t} | q, o_{i,<t})  -- 重要性采样比率
  Â_i = (r_i - mean(r)) / std(r)  -- 组内归一化优势
  ε -- clip 范围
  β -- KL 散度惩罚系数
  π_ref -- 参考策略（通常是 SFT 模型）
```

### 9.2 Tongyi DeepResearch 的 GRPO 定制化改进

Tongyi DeepResearch 在标准 GRPO 基础上做了多项关键改进，以适应 Agent 训练的特殊需求：

#### 9.2.1 Token 级策略梯度（Token-Level Policy Gradients）

**问题**：标准 GRPO 对每个 token 赋予相同的序列级优势值（即整个输出序列中每个 token 的优势 Â 相同）。但在 Agent 场景中，一个长序列中可能包含好的搜索决策和差的推理步骤，统一赋值不合理。

**改进**：Tongyi DeepResearch 采用 token 级梯度优化，更细粒度地调整每个 token 的梯度贡献。

```
标准 GRPO：
  每个 token 的优势 = Â_i（序列级，所有 token 相同）

Tongyi 改进：
  token 级策略梯度 = 在策略分布 π_θ(a|q) = Π_{t=1}^{N} π_θ(a_t | q, a_{<t}) 上
  更精细地计算每个 token 位置的梯度权重
  → 搜索动作 token、推理 token、输出 token 获得不同的梯度信号
```

#### 9.2.2 Leave-One-Out 优势估计

**问题**：标准组内归一化 `Â_i = (r_i - mean(r)) / std(r)` 在样本量小时方差大，且每个样本的 baseline 受自身影响。

**改进**：Leave-One-Out 方法 —— 计算第 i 个样本的优势时，排除第 i 个样本本身来计算均值和标准差。

```
标准 GRPO：
  baseline = mean({r_1, r_2, ..., r_G})
  （第 i 个样本的奖励参与了自己 baseline 的计算）

Leave-One-Out：
  baseline_i = mean({r_1, ..., r_{i-1}, r_{i+1}, ..., r_G})
  std_i = std({r_1, ..., r_{i-1}, r_{i+1}, ..., r_G})
  Â_i = (r_i - baseline_i) / std_i

优势：
  ├── 降低优势估计的方差
  ├── 在非平稳环境中更稳定
  └── 每个样本的评估更公正（不受自身影响）
```

#### 9.2.3 选择性负样本过滤（Selective Negative Sample Filtering）

**问题**：在 Agent RL 训练中，许多"失败"轨迹并非因为策略差，而是因为外部因素（搜索引擎返回差结果、网页加载失败等）。对这些低质量负样本计算梯度会引入噪声，降低训练稳定性。

**改进**：仔细过滤低质量负样本，只保留对学习有信息量的负样本。

```
采样 G 个轨迹 → 计算奖励

正样本（高奖励）：全部保留用于训练
负样本（低奖励）：
  ├── 分析失败原因
  ├── 过滤掉因外部因素（环境噪声）导致的失败
  ├── 保留因策略决策错误导致的失败
  └── 只用有信息量的负样本计算梯度

效果：
  ├── 显著提高训练稳定性
  ├── 避免模型从"不可控失败"中学到错误经验
  └── 在非平稳环境中尤为重要
```

#### 9.2.4 Clip-Higher 策略（促进探索）

**问题**：标准 PPO/GRPO 的 clip 机制 `clip(ρ, 1-ε, 1+ε)` 在正样本（Â > 0）时限制了概率增加的上限，在负样本（Â < 0）时限制了概率降低的下限。这对于 Agent 场景过于保守，模型可能陷入局部最优（总是搜索同类内容）。

**改进**：Clip-Higher 策略 —— 对正样本使用更宽松的 clip 上界，鼓励模型更大胆地探索成功策略。

```
标准 clip：   clip(ρ, 1-ε, 1+ε)     -- 对称 clip
Clip-Higher：clip(ρ, 1-ε, 1+ε')    -- ε' > ε，正方向更宽松

效果：
  ├── 鼓励模型在发现好策略时更激进地提高概率
  ├── 促进搜索策略的多样性
  └── 防止过早收敛到次优行为模式
```

#### 9.2.5 严格在线策略（Strictly On-Policy）

**关键设计**：Tongyi DeepResearch 采用**严格的在线策略训练**，即每次更新都使用当前策略新生成的轨迹，而非重用旧轨迹。

```
Off-Policy（离线策略）：
  采样轨迹 → 多次更新 → 数据复用率高 → 但分布偏移严重

On-Policy（在线策略）：
  采样轨迹 → 更新一次 → 丢弃旧轨迹 → 无分布偏移

Tongyi 选择严格 On-Policy 的原因：
  ├── Agent 环境高度非平稳（搜索引擎结果变化、网页内容更新）
  ├── 旧轨迹的动作分布与当前策略差异大
  ├── Off-Policy 修正（重要性采样）在长序列中方差爆炸
  └── 牺牲数据效率，换取训练稳定性
```

#### 9.2.6 二值奖励信号（Binary Reward）

```
奖励设计：
  r = 1  如果最终回答正确（事实准确、引用正确）
  r = 0  如果最终回答不正确

为什么用二值奖励而非连续奖励：
  ├── 简单、无需奖励模型（Reward Model）
  ├── 避免奖励黑客（Reward Hacking）
  ├── 在 GRPO 的组内比较框架中，二值奖励足以提供有效梯度
  │     （同一问题的多个采样中，正确回答获得正优势，错误回答获得负优势）
  └── 实践中效果好且稳定
```

### 9.3 Agent RL 训练的完整流程

```
GRPO Agent 训练循环：

for each training iteration:
    1. 采样一批问题 {q_1, q_2, ..., q_B}

    2. 对每个问题 q_i，用当前策略 π_θ 生成 G 个完整 Agent 轨迹
       每个轨迹包含：搜索查询 → 阅读网页 → 推理 → ... → 最终回答
       （轨迹长度可能达到数千 token，包含多个搜索-推理循环）

    3. 评估每个轨迹的奖励
       r_{i,j} = 1 如果最终回答正确，0 否则

    4. 对每组 G 个轨迹，计算 Leave-One-Out 优势
       Â_{i,j} = (r_{i,j} - baseline_{i,j}) / std_{i,j}

    5. 选择性过滤负样本
       移除因环境噪声导致的低质量失败轨迹

    6. 计算 token 级策略梯度，应用 Clip-Higher
       更新模型参数 θ

    7. 丢弃所有轨迹（严格 On-Policy）
```

---

## 十、Step-DeepResearch 论文解析

> 论文：[Step-DeepResearch Technical Report (arXiv:2512.20491)](https://arxiv.org/abs/2512.20491)
> 团队：StepFun Agent-Team（阶跃星辰）
> 开源：[GitHub - stepfun-ai/StepDeepResearch](https://github.com/stepfun-ai/StepDeepResearch)

### 10.1 核心理念："搜索不等于研究"

Step-DeepResearch 论文开篇提出了一个重要区分：

```
搜索（Search）                    研究（Research）
├── 目标：精确查询               ├── 目标：开放式探索
├── 输入：明确的查询语句         ├── 输入：模糊的研究问题
├── 输出：检索结果列表           ├── 输出：结构化研究报告
├── 优化：检索准确率             ├── 优化：综合分析质量
└── 单次交互                     └── 迭代过程
                                  ├── 意图分解
                                  ├── 规划
                                  ├── 有效工具使用
                                  ├── 跨源验证
                                  └── 报告综合
```

### 10.2 架构设计：单 Agent + ReAct

与 Tongyi DeepResearch 提出 IterResearch 新范式不同，Step-DeepResearch 采用**基于 ReAct 范式的单 Agent 架构**，通过推理-行动-反思的动态循环实现自主深度研究。

#### 工具集设计

| 工具 | 功能 | 说明 |
|------|------|------|
| `batch_web_surfer` | 批量网页搜索和浏览 | 支持批量操作，提高效率 |
| `file` | 文件读写和编辑 | 支持中间结果持久化 |
| `todo` | 任务状态管理 | 跟踪研究计划执行进度 |
| `shell` | 交互式命令执行 | 灵活的系统级操作 |

**设计理念**：精简但完备的工具集，覆盖搜索、持久化、规划管理、系统操作四个维度，支持完整的研究工作流。

### 10.3 核心创新：原子能力数据合成

这是 Step-DeepResearch 最大的技术贡献。

#### 10.3.1 原子能力分解

将复杂的深度研究能力分解为可独立训练的**原子能力（Atomic Capabilities）**：

```
深度研究
├── 规划能力（Planning）
│     ├── 问题分解
│     ├── 研究路径规划
│     └── 动态计划调整
│
├── 信息检索能力（Information Seeking）
│     ├── 查询构建
│     ├── 搜索策略选择
│     └── 信息提取
│
├── 反思与交叉验证能力（Reflection & Cross-Validation）
│     ├── 信息可靠性评估
│     ├── 多源信息交叉比对
│     └── 偏差检测与纠正
│
└── 专业报告生成能力（Professional Report Generation）
      ├── 结构化组织
      ├── 论据整合
      └── 引用管理
```

#### 10.3.2 训练目标的转变

```
传统 LLM 训练目标：
  "预测下一个 token"
  → 优化 P(token_t | token_1, ..., token_{t-1})

Step-DeepResearch 训练目标：
  "决定下一个原子动作"
  → 优化 P(atomic_action_t | context, plan, history)

实际效果：
  ├── 提高复杂环境中的鲁棒性
  ├── 增强跨任务泛化能力
  └── 减少传统合成数据集中的"能力缺失"问题
```

### 10.4 渐进式训练管线

```
阶段 1: Agentic Mid-Training（中期训练）
  ├── 基于原子能力合成的领域知识数据
  ├── 高层规划数据
  ├── 行为反思数据
  ├── 信息摘要数据
  └── 跨源验证数据

阶段 2: Supervised Fine-Tuning（监督微调）
  ├── 基于知识图谱的后训练数据合成
  ├── 基于专家轨迹的后训练数据合成
  └── 提高训练数据的信息密度和逻辑结构

阶段 3: Reinforcement Learning（强化学习）
  ├── 使用 Checklist-style Judger 作为奖励机制
  └── 在 ReAct 循环中进行在线策略优化
```

### 10.5 Checklist-style Judger 奖励设计

与 Tongyi DeepResearch 使用简单二值奖励不同，Step-DeepResearch 引入了**清单式评判器（Checklist-style Judger）**：

```
报告评估维度：
  □ 问题覆盖度：是否回答了所有子问题？
  □ 事实准确性：陈述是否有可靠来源支持？
  □ 信息深度：是否提供了足够的细节和分析？
  □ 逻辑连贯性：论证链是否完整和自洽？
  □ 引用质量：引用是否相关、准确、可追溯？
  □ 结构组织：报告结构是否清晰合理？

评分方式：
  每个维度独立评分 → 加权综合 → 最终奖励信号

优势：
  ├── 比二值奖励提供更丰富的梯度信号
  ├── 模型可以从不同维度的分数中学到具体改进方向
  └── 显著提高跨场景的鲁棒性
```

### 10.6 ADR-Bench：中文深度研究评估基准

Step-DeepResearch 团队开创性地建立了**面向中文领域的深度研究评估基准 ADR-Bench**（Application-driven Deep Research Benchmark）。

```
ADR-Bench 覆盖领域：
  ├── 商业研究（Commercial Research）
  ├── 政策分析（Policy Analysis）
  └── 软件工程（Software Engineering）

评估方法：
  ├── Elo 风格的评分协议
  ├── 多维度质量标准
  └── 自动化指标与人类感知有用性的对齐
```

### 10.7 性能与性价比

| 模型 | ResearchRubrics 得分 | 相对成本 |
|------|---------------------|---------|
| Step-DeepResearch (32B) | 61.42% | 1x |
| OpenAI DeepResearch | ~60% | >10x |
| Gemini DeepResearch | ~58% | >10x |

**关键成就**：在 ResearchRubrics 基准上排名第二，超越 OpenAI DeepResearch，成本仅为商业系统的 1/10。

### 10.8 Tongyi vs Step：关键差异对比

| 维度 | Tongyi DeepResearch | Step-DeepResearch |
|------|--------------------|--------------------|
| **Agent 范式** | IterResearch（新范式） + ReAct | 纯 ReAct 范式 |
| **上下文管理** | 轮次间工作区重构 | ReAct 单上下文 + todo 工具辅助 |
| **数据合成** | AgentFounder（知识图谱驱动） | 原子能力分解合成 |
| **奖励设计** | 二值奖励 + GRPO | Checklist-style Judger |
| **RL 算法** | 定制化 GRPO | 未详述具体算法 |
| **模型规模** | 30.5B (3.3B 激活, MoE) | 32B |
| **中文评估** | BrowseComp-ZH | ADR-Bench（新建） |
| **核心优势** | MDP 建模 + 端到端 RL | 原子能力 + 成本效率 |

---

## 十一、构建深度研究系统的实践洞察

### 11.1 架构选择

```
场景分析：

如果你有强大的 RL 基础设施：
  → 选择 IterResearch 范式
  → MDP 建模 + GRPO 端到端训练
  → 最优上限高，但工程复杂度也高

如果你追求快速落地：
  → 选择 ReAct 范式
  → SFT 训练 + 简单 RL
  → 工程简单，上线快

如果你没有训练能力：
  → 直接使用开源模型（Tongyi-DeepResearch-30B-A3B）
  → 配合 Qwen-Agent 框架
  → 零训练成本
```

### 11.2 上下文管理策略

这是深度研究系统最关键的工程挑战之一：

```
策略 1: 全量保留（ReAct 原生）
  ├── 优点：信息无损
  ├── 缺点：上下文爆炸、推理质量下降
  └── 适用：轮次少（<10 轮）的简单任务

策略 2: 滑动窗口
  ├── 优点：实现简单
  ├── 缺点：可能丢失关键早期信息
  └── 适用：中等复杂度任务

策略 3: 中央报告 + 工作区重构（IterResearch）
  ├── 优点：信息压缩有效、推理质量稳定
  ├── 缺点：报告更新质量依赖模型摘要能力
  └── 适用：长程复杂任务（10-50+ 轮）

策略 4: 分层记忆（Hierarchical Memory）
  ├── 短期记忆：当前轮次上下文
  ├── 中期记忆：结构化知识库（Key-Value 形式）
  ├── 长期记忆：压缩后的历史总结
  └── 适用：需要在多个任务间共享知识
```

### 11.3 搜索策略优化

```
初级：固定查询模板
  query = f"关于{topic}的{aspect}"

中级：动态查询生成
  query = LLM(context + "生成最有价值的下一个搜索查询")

高级：多策略搜索
  ├── 广度搜索：覆盖问题的多个方面
  ├── 深度搜索：对特定方面深入挖掘
  ├── 验证搜索：交叉验证已有发现
  └── 补充搜索：填补知识空白

最佳实践：
  ├── 让 RL 学习搜索策略（如 Tongyi 的做法）
  ├── 而非手工设计搜索规则
  └── 模型自动学会"什么时候搜、搜什么、什么时候停"
```

### 11.4 停止决策

何时结束研究是 Deep Research 系统的关键决策点：

```
启发式方法：
  ├── 固定轮次上限
  ├── 信息增益阈值（新搜索不再带来新发现时停止）
  └── 置信度阈值（模型对当前报告的自信度足够高时停止）

RL 学习的停止策略：
  ├── 模型通过大量训练轨迹学会最优停止时机
  ├── 过早停止 → 报告不完整 → 低奖励
  ├── 过晚停止 → 浪费计算 → 效率惩罚
  └── RL 自动平衡完整性与效率
```

### 11.5 评估方法

```
自动评估：
  ├── 事实准确性：与 ground truth 比较（适用于有标准答案的 benchmark）
  ├── 引用验证：检查引用链接是否有效、内容是否匹配
  └── 格式检查：报告结构是否完整

LLM-as-Judge：
  ├── 用强模型（如 GPT-4o、Claude）评估报告质量
  ├── 多维度评分（类似 Step 的 Checklist Judger）
  └── 可以处理开放式问题的评估

人工评估：
  ├── Elo 评分（成对比较）
  ├── 多维度打分
  └── 最准确，但成本最高

推荐组合：
  开发阶段 → 自动评估 + LLM-as-Judge
  发布前 → 人工评估验证
```

### 11.6 工程实践要点

```
1. 搜索引擎选择
   ├── 可配置多个搜索后端（Google、Bing、Perplexity 等）
   ├── 根据查询类型选择最佳搜索引擎
   └── 搜索结果缓存以减少 API 调用

2. 网页内容提取
   ├── 使用 Playwright/Puppeteer 处理动态网页
   ├── HTML → Markdown 转换（保留关键结构）
   └── 处理反爬虫机制

3. 成本控制
   ├── MoE 模型（如 Tongyi 的 30.5B/3.3B）大幅降低推理成本
   ├── 上下文管理减少每步的 token 消耗
   ├── 搜索结果缓存减少 API 调用
   └── 早停策略避免无效计算

4. 可靠性
   ├── 搜索失败重试机制
   ├── 网页加载超时处理
   ├── 异常轨迹检测和恢复
   └── 中间结果持久化（防止长任务中断丢失进度）

5. 多语言支持
   ├── 查询翻译（将中文查询翻译为英文搜索，或反之）
   ├── 多语言网页内容理解
   └── 最终报告的目标语言控制
```

### 11.7 开源复现路线图

对于想要从零构建 Deep Research 系统的团队：

```
阶段 1（1-2 周）：基于提示工程的 MVP
  ├── 使用 Qwen3/Tongyi-DeepResearch 开源模型
  ├── ReAct 格式 Prompt
  ├── 接入搜索 API
  └── 简单的循环调度逻辑

阶段 2（2-4 周）：增强版
  ├── 实现 IterResearch 的工作区重构
  ├── 添加中央报告管理
  ├── 优化搜索策略
  └── 添加引用管理

阶段 3（1-3 月）：SFT 训练
  ├── 收集/合成 Agent 轨迹数据
  ├── 在 ReAct + IterResearch 两种格式上微调
  ├── 评估基准搭建
  └── 迭代优化

阶段 4（3-6 月）：RL 训练
  ├── 搭建 GRPO 训练基础设施
  ├── 设计奖励函数
  ├── 在线策略训练
  └── 端到端优化 Agent 行为
```

---

## 参考资料

- [Tongyi DeepResearch Technical Report (arXiv:2510.24701)](https://arxiv.org/abs/2510.24701)
- [Tongyi DeepResearch Blog](https://tongyi-agent.github.io/blog/introducing-tongyi-deep-research/)
- [Step-DeepResearch Technical Report (arXiv:2512.20491)](https://arxiv.org/abs/2512.20491)
- [Alibaba-NLP/DeepResearch GitHub](https://github.com/Alibaba-NLP/DeepResearch)
- [StepFun/StepDeepResearch GitHub](https://github.com/stepfun-ai/StepDeepResearch)
- [DeepSeekMath GRPO (arXiv:2402.03300)](https://arxiv.org/abs/2402.03300)
- [Explaining Tongyi DeepResearch (Towards AI)](https://towardsai.net/p/machine-learning/explaining-tongyi-deepresearch)
- [GRPO Illustrated Breakdown](https://epichka.com/blog/2025/grpo/)
- [Tongyi-DeepResearch-30B-A3B on HuggingFace](https://huggingface.co/Alibaba-NLP/Tongyi-DeepResearch-30B-A3B)
