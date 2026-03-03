# 05 - 开源模型 Tech Report 解读

> 精选 7 个代表性开源模型的技术报告解读，覆盖 MoE 工程、推理训练、多模态、完全开源四大方向。

## 选择逻辑

| 模型 | 为什么选它 | 核心看点 |
|------|-----------|---------|
| [DeepSeek-V3](./DeepSeek-V3.md) | $5.57M 训练出 GPT-4o 级别模型 | MLA, 无辅助损失负载均衡, FP8 训练 |
| [DeepSeek-R1](./DeepSeek-R1.md) | 纯 RL 涌现推理能力的里程碑 | R1-Zero, GRPO, 四阶段流水线, 蒸馏 |
| [Kimi K2.5](./Kimi-K2.5.md) | 万亿参数 MoE + 多模态 + Agent Swarm | PARL, MuonClip, 联合视觉训练 |
| [Step 3.5 Flash](./Step-3.5-Flash.md) | 11B 激活达到前沿推理水平 | 极致效率 MoE |
| [GLM-5](./GLM-5.md) | 国产芯片训练的前沿模型 | 昇腾 910B, Agent 能力 |
| [OLMo 3](./OLMo-3.md) | 最完整的全开源训练流水线 | 数据→代码→模型全开源 |
| [Molmo 2](./Molmo-2.md) | 开源多模态 SOTA | 视频理解, 视觉定位 |

## 模型参数对比

| 模型 | 总参数 | 激活参数 | 架构 | 上下文 | 开源协议 |
|------|--------|----------|------|--------|---------|
| DeepSeek-V3 | 671B | 37B | MoE (256+1 experts) | 128K | 开源 |
| DeepSeek-R1 | 671B | 37B | MoE (基于 V3) | 128K | MIT |
| Kimi K2.5 | 1T | 32B | MoE (384 experts) | 256K | 开源 |
| Step 3.5 Flash | 197B | 11B | MoE (288+1 experts) | 256K | 开源 |
| GLM-5 | 744B | 44B | MoE | 200K | MIT |
| OLMo 3 | 32B | 32B | Dense | 长上下文 | Apache 2.0 |
| Molmo 2 | 8B | 8B | Dense + Vision | - | Apache 2.0 |

## 阅读建议

```
想了解 MoE 训练工程？     → DeepSeek-V3 (MLA + FP8) 或 Kimi K2.5 (最大规模) 或 Step 3.5 Flash (最高效率)
想了解 RL 训练推理能力？   → DeepSeek-R1 (GRPO + 四阶段流水线)
想了解完整预训练流水线？   → OLMo 3 (从数据到代码全开源)
想了解多模态训练？        → Kimi K2.5 (文本-视觉联合) + Molmo 2 (视觉定位)
想了解 Agent 训练？       → Kimi K2.5 (PARL) + GLM-5 (SWE-bench SOTA)
想了解国产化训练？        → GLM-5 (10万张昇腾 910B)
```
