# 01 - 预训练工程

> 预训练是大模型能力的基石。本章涵盖数据工程、模型架构、训练基础设施与推理优化四大板块。

## 总览

预训练工程可分为四个核心子方向：

```
预训练工程
├── 数据工程     → 数据采集、清洗、去重、质量过滤、合成数据
├── 模型架构     → Transformer 变体、MoE、MLA、SSM、长上下文
├── 训练基础设施  → 并行策略、优化器、混合精度、容错
└── 推理优化     → 量化、KV Cache、推测解码、Serving 框架
```

## 子文档

- [数据工程](./数据工程.md) - 从原始网页到高质量 Token 的完整流水线
- [模型架构](./模型架构.md) - MoE、MLA、DiT 等架构创新
- [训练基础设施](./训练基础设施.md) - 并行策略、优化器、Scaling Laws
- [推理优化](./推理优化.md) - 量化、KV Cache 优化、Serving 框架

## 关键数字

| 模型 | 总参数 | 激活参数 | 预训练数据量 | 训练成本 |
|------|--------|----------|-------------|---------|
| DeepSeek-V3 | 671B | 37B | 14.8T tokens | ~$5.6M |
| Kimi K2 (Base) | 1T | 32B | 15.5T tokens | - |
| Kimi K2.5 | 1T | 32B | +15T vision-text | - |
| GLM-5 | 744B | 44B | 28.5T tokens | - |
| Step 3.5 Flash | 197B | 11B | - | - |
| Llama 3.1 405B | 405B | 405B (Dense) | 15T tokens | - |
| OLMo 3 32B | 32B | 32B (Dense) | ~6T tokens | - |

## 核心参考

- [Llama 3 技术报告](https://arxiv.org/abs/2407.21783) - 最全面的全栈 LLM 工程参考
- [DeepSeek-V3 技术报告](https://arxiv.org/abs/2412.19437) - MoE + FP8 + MLA 工程典范
- [OLMo 3 博客](https://allenai.org/blog/olmo3) - 完全开源的预训练流水线
