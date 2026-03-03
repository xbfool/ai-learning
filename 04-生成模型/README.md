# 04 - 生成模型

> 图像生成、视频生成、多模态统一模型——生成式 AI 在 2025-2026 进入实用化阶段。

## 总览

```
生成模型
├── 图像生成
│   ├── 扩散模型 (Diffusion) → FLUX.2, SD 3.5, HiDream
│   ├── 自回归模型 (AR)     → GPT-4o Images, Emu3
│   └── 混合模型           → GLM-Image (AR + Diffusion)
│
├── 视频生成
│   ├── DiT + Flow Matching → Sora 2, Wan 2.2, HunyuanVideo
│   ├── 长视频/叙事        → Sora 2 (25s), Kling 3.0
│   └── 音频同步           → Sora 2, Veo 3.1
│
└── 多模态统一
    ├── 理解 + 生成统一    → GPT-4o, Gemini, Transfusion
    ├── 解耦路径           → Janus (DeepSeek)
    └── 世界模型           → Cosmos (NVIDIA), Genie 3
```

## 关键趋势

1. **DiT (Diffusion Transformer) 取代 U-Net** → 所有 SOTA 生成模型都用 Transformer
2. **Flow Matching 取代 DDPM** → 更直的轨迹 → 更少采样步骤
3. **自回归图像生成崛起** → GPT-4o 证明 AR 可以高质量生图
4. **视频生成走向实用** → 25 秒连贯视频 + 原生音频
5. **统一多模态模型** → 一个模型同时理解和生成文字、图像、视频、音频

## 子文档

- [图像生成](./图像生成.md) - FLUX.2、扩散 vs 自回归、架构演进
- [视频生成](./视频生成.md) - Sora 2、Wan 2.2、Veo 3.1、开源生态
- [多模态统一模型](./多模态统一模型.md) - 从分离到统一的技术路线

## 生态格局 (2026 初)

### 图像生成
| 梯队 | 模型 | 特点 |
|------|------|------|
| 闭源顶尖 | Midjourney v7, GPT-4o Images | 美学/指令遵循 |
| 开源顶尖 | FLUX.2 [dev], HiDream-I1 | 可本地部署 |
| 特色模型 | Recraft V4, Ideogram | 设计/文字渲染 |
| 国产开源 | GLM-Image, Kolors | 中文/多语言 |

### 视频生成
| 梯队 | 模型 | 特点 |
|------|------|------|
| 闭源顶尖 | Sora 2, Veo 3.1 | 最高质量 |
| 中国竞争者 | Kling 3.0, Seedance 2.0 | 快速迭代 |
| 开源标杆 | Wan 2.2, HunyuanVideo | 可本地部署 |
| 新兴力量 | Luma Ray2, MiniMax | 各有特色 |

### 行业数据
- 企业生产部署中位数使用 **14 个不同模型**
- 开源模型因可定制性在企业生产中越来越受欢迎
- 用例分布：广告创意 > 电商产品图 > 游戏素材 > 社交媒体内容
