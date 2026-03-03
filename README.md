# AI 技术全景 2025-2026 总论

> 最后更新：2026-03-03
> 本文档系统梳理当前 AI 领域的核心技术方向，采用自上而下的分层结构，从宏观全景到具体技术细节。

## 一、全景概览

当前 AI（特别是大语言模型 LLM）领域可划分为以下六大技术板块：

```
AI 技术全景
├── 1. 预训练工程        → 数据、架构、基础设施、Scaling Laws
├── 2. 后训练与对齐      → SFT、RLHF、GRPO、RL 训练框架
├── 3. 推理与 Agent      → 推理模型、DeepResearch、Agent 系统
├── 4. 生成模型          → 图像生成、视频生成、多模态统一模型
├── 5. 推理优化与部署    → 量化、KV Cache、推测解码、Serving 框架
└── 6. 开源模型解读      → Kimi K2.5, Step 3.5 Flash, GLM-5, OLMo 3, Molmo 2
```

## 二、2025-2026 年度核心趋势

### 趋势 1：推理时计算 (Test-Time Compute) 成为新的 Scaling 维度
- OpenAI o1/o3/o4、DeepSeek-R1、Kimi K2.5、Step 3.5 Flash 等模型证明：**推理阶段的计算量**可以替代模型规模
- 核心手段：Chain-of-Thought RL、长思考 (Extended Thinking)、搜索/验证循环
- 关键论文：[Scaling LLM Test-Time Compute](https://arxiv.org/abs/2408.03314)

### 趋势 2：RL 成为后训练的主导范式
- GRPO（Group Relative Policy Optimization）取代 PPO 成为主流算法
  - 无需 Critic 模型，大幅简化训练流程
  - DeepSeek-R1 首创，后被 Qwen、Kimi、Step 等广泛采用
- RL 训练从单轮对话扩展到**多步推理**和**Agent 行为**
- 规则/可验证奖励（数学答案验证、代码执行）逐步取代学习型奖励模型

### 趋势 3：Agent 化成为核心应用方向
- Deep Research 类产品（Google、OpenAI、Qwen、Step）成为第一个大规模落地的 Agent 应用
- Agent 训练从 Prompt Engineering 转向 **端到端 RL 训练**
- Kimi K2.5 的 Agent Swarm（PARL）代表了并行 Agent 编排的新方向
- MCP (Model Context Protocol) 标准化 Agent 工具调用

### 趋势 4：MoE 架构统治万亿参数级模型
- DeepSeek-V3 (671B/37B active)、Kimi K2.5 (1T/32B active)、GLM-5 (744B/44B active)、Step 3.5 Flash (197B/11B active)
- 关键创新：无辅助损失负载均衡、细粒度专家、共享专家、MLA
- MoE 使训练成本降低一个数量级（DeepSeek-V3 仅花费 $5.6M）

### 趋势 5：视频生成从技术演示走向实用
- 2025 下半年至 2026 初，Sora 2、Kling 3.0、Seedance 2.0、Veo 3.1 密集发布
- 原生音频生成成为标配
- 开源模型（Wan 2.2、HunyuanVideo）快速追赶
- 企业生产环境使用中位数 **14 个**不同模型

### 趋势 6：完全开源 (Fully Open) 成为竞争力
- AI2 的 OLMo 3 + Molmo 2：从数据到代码到模型全部开源
- 中国实验室主导开源生成模型：HunyuanVideo、Wan、CogVideoX、Janus
- 开源 RL 框架：veRL (字节)、ROLL (淘天)、AReaL (蚂蚁) 三足鼎立

## 三、文档目录

| 目录 | 内容 | 关键词 |
|------|------|--------|
| [01-预训练工程](./01-预训练工程/) | 数据工程、模型架构、训练基础设施、推理优化 | FineWeb, MoE, MLA, Megatron, vLLM |
| [02-后训练与对齐](./02-后训练与对齐/) | SFT、RL 算法、RL 训练框架 | GRPO, DPO, veRL, ROLL, AReaL |
| [03-推理与Agent](./03-推理与Agent/) | 推理模型、DeepResearch、Agent 系统 | DeepSeek-R1, Qwen DeepResearch, IterResearch |
| [04-生成模型](./04-生成模型/) | 图像生成、视频生成、多模态统一 | FLUX.2, Sora 2, Wan 2.2, DiT, Flow Matching |
| [05-开源模型Tech-Report](./05-开源模型Tech-Report/) | 具体模型技术报告解读 | Kimi K2.5, Step 3.5 Flash, GLM-5, OLMo 3, Molmo 2 |
| [06-推荐阅读清单](./06-推荐阅读清单.md) | 按主题分类的必读论文 | 综合阅读指南 |

## 四、技术栈全景图

```
┌──────────────────── AI 模型开发全流程 ────────────────────┐
│                                                            │
│  数据层          架构层         训练层        部署层         │
│  ┌─────────┐   ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │FineWeb  │   │Transformer│  │预训练    │  │vLLM      │   │
│  │DCLM     │   │MoE       │  │ (15T+    │  │SGLang    │   │
│  │Dolma 3  │   │MLA/GQA   │  │  tokens) │  │TRT-LLM   │   │
│  │合成数据  │   │DiT       │  │          │  │          │   │
│  └────┬────┘   │SSM/Mamba │  │SFT       │  │量化      │   │
│       │        └────┬─────┘  │ (高质量  │  │ GPTQ/AWQ │   │
│       │             │        │  少量数据)│  │ FP8      │   │
│       │             │        │          │  │          │   │
│       └─────────────┼────────│RL/GRPO   │  │推测解码  │   │
│                     │        │ (推理/   │  │ Medusa   │   │
│                     │        │  Agent)  │  │ Eagle    │   │
│                     │        └──────────┘  └──────────┘   │
│                                                            │
│  并行策略: TP + PP + DP + CP + EP (Megatron / DeepSpeed)   │
└────────────────────────────────────────────────────────────┘
```

## 五、快速导航

**想了解预训练怎么做？** → [01-预训练工程/README.md](./01-预训练工程/README.md)
**想了解 RL 训练怎么做？** → [02-后训练与对齐/README.md](./02-后训练与对齐/README.md)
**想了解 Agent 和 DeepResearch？** → [03-推理与Agent/README.md](./03-推理与Agent/README.md)
**想了解图像/视频生成？** → [04-生成模型/README.md](./04-生成模型/README.md)
**想看具体模型技术报告？** → [05-开源模型Tech-Report/README.md](./05-开源模型Tech-Report/README.md)
**想要一份阅读清单？** → [06-推荐阅读清单.md](./06-推荐阅读清单.md)
