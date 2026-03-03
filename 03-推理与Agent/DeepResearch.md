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
