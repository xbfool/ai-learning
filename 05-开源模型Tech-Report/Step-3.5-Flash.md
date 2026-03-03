# Step 3.5 Flash 技术报告解读

> 阶跃星辰 (StepFun) 的极致效率模型：仅 11B 激活参数达到前沿推理水平。

## 基本信息

- **开发者**：阶跃星辰 (StepFun)
- **技术博客**：[static.stepfun.com/blog/step-3.5-flash](https://static.stepfun.com/blog/step-3.5-flash/)
- **HuggingFace**：[stepfun-ai/Step-3.5-Flash](https://huggingface.co/stepfun-ai/Step-3.5-Flash)
- **发布时间**：2026 年 2 月

## 架构

| 参数 | 值 |
|------|-----|
| 总参数 | 196.81B |
| 激活参数 | ~11B |
| 层数 | 45 |
| 隐藏维度 | 4,096 |
| 路由专家数 | 288 |
| 共享专家数 | 1 |
| SWA 比例 | 3:1 (Sliding Window Attention) |
| 上下文长度 | 256K (输入) |

### 极致效率 MoE
- 288 个路由专家 + 1 个共享专家
- 仅 ~11B 激活参数（总参数的 ~5.6%）
- 这是目前最高效的前沿推理模型之一

### SWA 3:1 比例
- Sliding Window Attention (滑动窗口注意力)
- 3:1 比例：3 层滑动窗口注意力 : 1 层全注意力
- 减少长序列的注意力计算量，同时保持全局信息流动

## 性能 ★★★

Step 3.5 Flash 在多个推理基准上达到了令人惊叹的分数：

| 基准 | 得分 | 说明 |
|------|------|------|
| **AIME 2025** | **99.8** | 几乎满分 |
| **HMMT 2025 Nov** | **98.0** | 顶级数学竞赛 |
| **IMOAnswerBench** | **86.7** | IMO 级别 |
| **ARC-AGI-1** | **56.5** | 抽象推理 |
| ResearchRubrics | 65.27 | 研究能力 |

**AIME 2025 达到 99.8 分，仅用 11B 激活参数**——这证明了高效 MoE + RL 训练的巨大潜力。

## 推理性能

- **吞吐量**：100-300 tok/s（峰值 350 tok/s，代码任务）
- 达到**实时响应**级别
- 适合 Agent 场景中需要快速决策的需求

## 部署支持

支持主流推理框架：
- vLLM
- SGLang
- HuggingFace Transformers
- llama.cpp (本地推理)

## 配套：Step DeepResearch

- [GitHub - stepfun-ai/StepDeepResearch](https://github.com/stepfun-ai/StepDeepResearch)
- 基于 Step 3.5 Flash 构建的 Deep Research 系统
- 开源

## 核心贡献总结

1. **极致参数效率**：197B 总参数 / 11B 激活 → 5.6% 激活率
2. **前沿推理水平**：AIME 99.8%，11B 激活超越许多更大模型
3. **实时推理**：100-350 tok/s 吞吐
4. **SWA 3:1 架构**：高效的滑动窗口注意力配比
5. **完全开源**：权重 + 推理代码

## 值得深入研究的部分
- 288 专家的路由和负载均衡策略
- SWA 3:1 比例的消融实验
- 推理 RL 训练的具体方案
- 如何在 11B 激活中达到如此强的推理能力
