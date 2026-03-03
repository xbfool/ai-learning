# AI 技术全景 2025-2026 总论

> 最后更新：2026-03-03
> 本文档系统梳理当前 AI 领域的核心技术方向，采用自上而下的分层结构，从宏观全景到具体技术细节。

## 一、全景概览

当前 AI（特别是大语言模型 LLM）领域可划分为以下八大板块：

```
AI 技术全景
├── 1. 预训练工程        → 数据、架构、基础设施、Scaling Laws
├── 2. 后训练与对齐      → SFT、RLHF、GRPO、RL 框架、评估基准、AI 安全
├── 3. 推理与 Agent      → 推理模型、DeepResearch、Agent 系统、MCP、Coding Agent
├── 4. 生成模型          → 图像生成、视频生成、多模态统一模型
├── 5. 开源模型解读      → DeepSeek-V3/R1, Kimi K2.5, Step 3.5 Flash, GLM-5, OLMo 3, Molmo 2
├── 6. 语音与音频        → TTS、ASR、音乐生成、语音 Agent、视频原生音频
├── 7. 推荐阅读清单      → 按主题分类的必读论文
└── 8. 学习路线与实战    → 零基础入门、Agent 开发路径、本地 AI 体验项目
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
- Coding Agent (Claude Code, Cursor, Devin) 改变软件开发方式
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

### 趋势 7：语音 AI 爆发
- 原生语音对话 (GPT-4o Voice, Gemini Live) 延迟降至 232ms
- 开源 TTS 全面突破：CosyVoice 2、F5-TTS、Fish Speech 达到人类级质量
- 视频原生音频成为标配：Sora 2、Veo 3、Kling 3.0 同步生成视频和音频

## 三、文档目录

| 目录 | 内容 | 关键词 |
|------|------|--------|
| [01-预训练工程](./01-预训练工程/) | 数据工程、模型架构、训练基础设施、推理优化 | FineWeb, MoE, MLA, Megatron, vLLM |
| [02-后训练与对齐](./02-后训练与对齐/) | SFT、RL 算法、RL 训练框架、评估基准、AI 安全 | GRPO, DPO, veRL, MMLU, SWE-bench, Constitutional AI |
| [03-推理与Agent](./03-推理与Agent/) | 推理模型、DeepResearch、Agent 系统 | DeepSeek-R1, MCP, Coding Agent, Agent Swarm |
| [04-生成模型](./04-生成模型/) | 图像生成、视频生成、多模态统一 | FLUX.2, Sora 2, Wan 2.2, DiT, Flow Matching |
| [05-开源模型Tech-Report](./05-开源模型Tech-Report/) | 7 个模型技术报告解读 | DeepSeek-V3/R1, Kimi K2.5, Step 3.5 Flash, GLM-5, OLMo 3, Molmo 2 |
| [06-推荐阅读清单](./06-推荐阅读清单.md) | 按主题分类的必读论文 | 综合阅读指南 |
| [07-语音与音频模型](./07-语音与音频模型/) | TTS、ASR、音乐生成、语音 Agent | CosyVoice 2, Whisper, Suno, Realtime API |
| [08-学习路线与实战项目](./08-学习路线与实战项目/) | 零基础入门、Agent 开发路径、本地体验 | Ollama, LangGraph, MCP, ComfyUI, Unsloth |

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
│                                                            │
│  音频层: TTS (CosyVoice/F5-TTS) │ ASR (Whisper/SenseVoice)│
│  安全层: Constitutional AI │ RLHF │ Red Team │ SAE 可解释性 │
└────────────────────────────────────────────────────────────┘
```

## 五、快速导航

**想了解预训练怎么做？** → [01-预训练工程/README.md](./01-预训练工程/README.md)
**想了解 RL 训练怎么做？** → [02-后训练与对齐/README.md](./02-后训练与对齐/README.md)
**想了解 Agent 和 DeepResearch？** → [03-推理与Agent/README.md](./03-推理与Agent/README.md)
**想了解图像/视频生成？** → [04-生成模型/README.md](./04-生成模型/README.md)
**想看具体模型技术报告？** → [05-开源模型Tech-Report/README.md](./05-开源模型Tech-Report/README.md)
**想要一份阅读清单？** → [06-推荐阅读清单.md](./06-推荐阅读清单.md)
**想了解语音和音频 AI？** → [07-语音与音频模型/README.md](./07-语音与音频模型/README.md)
**想了解评估基准和 AI 安全？** → [02-后训练与对齐/评估基准与方法论.md](./02-后训练与对齐/评估基准与方法论.md)、[AI安全与对齐](./02-后训练与对齐/AI安全与对齐.md)
**零基础不知从哪开始？** → [08-学习路线与实战项目/零基础入门指南.md](./08-学习路线与实战项目/零基础入门指南.md)
**想学 Agent 开发？** → [08-学习路线与实战项目/Agent开发学习路径.md](./08-学习路线与实战项目/Agent开发学习路径.md)
**想在自己电脑上跑 AI？** → [08-学习路线与实战项目/本地AI体验项目.md](./08-学习路线与实战项目/本地AI体验项目.md)

---

## 致谢

特别感谢好友**魏叔叔**在 AI 学习方向上的悉心指引与启发。从技术路线的梳理到学习节奏的把握，许多关键思路都源自他的点拨。能在这个浪潮中少走弯路，离不开良师益友的引路。
