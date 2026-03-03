# 音乐生成与语音 Agent

> 音乐生成走向实用化，语音 Agent 基础设施成熟，视频原生音频成为标配。

---

## 一、音乐与音频生成

### 1.1 产品格局

```
音乐生成竞争格局 (2026 初)：

商业闭源
├── Suno v4.5+     → 市场领导者，44.1kHz 录音棚级
├── Udio            → UMG 授权，版权合规路线
└── YouTube Music AI → Google 内部能力

开源
├── ACE-Step v1.5   → 扩散模型，A100 上 <2 秒/首
└── Stable Audio Open → 341M 参数，移动端可运行

音频/音效
├── ElevenLabs Sound Effects → 商业 API
└── Stable Audio 2.5         → 专业级音效
```

### 1.2 Suno

> 文本/歌词到音乐的市场领导者。

| 项目 | 详情 |
|------|------|
| **公司** | Suno Inc. (美国) |
| **版本线** | v4 (2024.11.19) → v4.5 (2025.05.01) → v4.5+ (2025.07) |
| **音质** | 44.1kHz 录音棚级 |
| **能力** | 1,200+ 音乐风格；最长 8 分钟生成 (v4.5)；自然的人声演唱 (vibrato, phrasing) |
| **v4.5+ 新功能** | 为现有曲目添加人声/乐器；智能提示优化；音轨分离编辑；旧曲重新混音 |
| **收购** | 2025.06 收购 WavTool (浏览器 DAW)：VST 插件、精确编辑、录音、AI MIDI 生成 |
| **链接** | [suno.com](https://suno.com/home), [v4 博客](https://suno.com/blog/v4), [v4.5 博客](https://suno.com/blog/introducing-v4-5) |

### 1.3 Udio

| 项目 | 详情 |
|------|------|
| **公司** | Udio (美国) |
| **状态** | 与环球音乐 (UMG) 达成授权协议，解决版权纠纷 |
| **特点** | 文本描述生成逼真人声和乐器；音轨分离；多语言歌词 |
| **新方向** | 新模型使用 UMG 授权训练数据，全授权平台计划 2026 Q2 上线 |
| **链接** | [udio.com](https://www.udio.com/) |

### 1.4 ACE-Step (开源) ★★★

> 扩散模型路线，开源音乐生成里程碑。

| 项目 | 详情 |
|------|------|
| **团队** | ACE Studio + StepFun (中国合作) |
| **发布** | v1 (2025.05 开源), v1.5 (2025 下半年) |
| **参数** | 3.5B (v1) |
| **架构** | 扩散模型 + Sana DCAE (深度压缩自编码器) + 轻量线性 Transformer；MERT 和 m-hubert 语义对齐 (REPA) |
| **速度** | v1: A100 上 20 秒生成 4 分钟音乐 (比 LLM 基线快 15 倍)；v1.5: A100 < 2 秒/首，RTX 3090 < 10 秒 |
| **语言** | 19 种语言 (英中俄西日德法葡意韩等) |
| **功能** | 语音克隆、歌词编辑、混音、歌词转人声、演唱转伴奏 |
| **开源** | 是 |
| **链接** | [GitHub v1](https://github.com/ace-step/ACE-Step), [GitHub v1.5](https://github.com/ace-step/ACE-Step-1.5), [论文](https://arxiv.org/abs/2506.00045), [HuggingFace](https://huggingface.co/ACE-Step/ACE-Step-v1-3.5B) |

### 1.5 Stable Audio

| 项目 | 详情 |
|------|------|
| **公司** | Stability AI (英国) |
| **版本** | Open 1.0 (开源, 47 秒)；2.5 (商业, 3 分钟)；Open Small (2025.05, 341M 参数, 移动端) |
| **Stable Audio 2.5** | 专业级；复杂多段结构 (前奏/发展/尾声)；音频修复 (inpainting)；H100 上 <2 秒生成 |
| **Open Small** | 与 Arm 合作；341M 参数可在 Arm CPU 上运行；8 秒内在移动端生成 11 秒 44.1kHz 立体声 |
| **适用** | 音效、鼓点、乐器 riff、环境音、foley 效果 |
| **链接** | [开源版](https://stability.ai/news/introducing-stable-audio-open), [HuggingFace](https://huggingface.co/stabilityai/stable-audio-open-1.0) |

---

## 二、语音 Agent 与实时语音

### 2.1 技术架构对比

```
语音 Agent 两种架构：

架构 A：串联式 (Cascaded)
  ┌────────┐    ┌──────┐    ┌────────┐
  │ ASR    │───►│ LLM  │───►│ TTS    │
  │(语音→文)│    │(思考)│    │(文→语音)│
  └────────┘    └──────┘    └────────┘
  延迟：1-3 秒 (三段延迟叠加)
  优势：模块化、可替换组件
  代表：早期语音助手、Alexa、Siri

架构 B：原生端到端 (Native)  ★★★ 当前方向
  ┌──────────────────────────┐
  │    原生多模态模型          │
  │  语音 → 理解+思考 → 语音  │
  └──────────────────────────┘
  延迟：232ms (GPT-4o)
  优势：保留语气/情感、更自然
  代表：GPT-4o Voice, Gemini Live
```

### 2.2 OpenAI Realtime API

> 语音 Agent 的基础设施标准。

| 项目 | 详情 |
|------|------|
| **公司** | OpenAI (美国) |
| **GA 日期** | 2025 年 8 月 28 日 |
| **模型** | gpt-realtime (语音到语音)；gpt-realtime-mini (成本/延迟优化) |
| **核心能力** | 低延迟双向音频流；远程 MCP 服务器支持；图像输入；SIP 电话呼叫 |
| **语音** | 2 个专属语音 (Cedar, Marin) |
| **特色** | 复杂指令遵循、精准工具调用、自然表达性语音 |
| **链接** | [公告](https://openai.com/index/introducing-gpt-realtime/), [文档](https://platform.openai.com/docs/guides/realtime), [语音 Agent 指南](https://developers.openai.com/api/docs/guides/voice-agents/) |

**Realtime API 开发模式**：

```
WebSocket 连接流程：

客户端                              服务器
  │                                   │
  │──── WebSocket 连接 ──────────────►│
  │                                   │
  │◄──── session.created ────────────│
  │                                   │
  │──── 音频流 (持续发送) ────────────►│
  │                                   │
  │◄──── 中间转录 ──────────────────│
  │◄──── 函数调用请求 ──────────────│
  │                                   │
  │──── 函数结果 ──────────────────►│
  │                                   │
  │◄──── 音频响应流 ────────────────│
  │                                   │

特色：
  - 支持用户随时打断 (barge-in)
  - 服务端自动检测发言结束 (VAD)
  - 可同时发送音频和接收音频 (全双工)
```

### 2.3 GPT-4o 原生语音模式

| 项目 | 详情 |
|------|------|
| **发布** | 2024 年 5 月 (GPT-4o 发布)；Advanced Voice Mode 2024-2025 逐步推出 |
| **延迟** | 最快 232ms，平均 320ms 响应 |
| **核心特色** | 原生语音到语音 (无中间 ASR/TTS)；捕捉非语言线索 (语速/语调)；支持用户打断；可以唱歌；情感表达 (笑声/哭泣/共情/讽刺)；角色语音模仿 |
| **链接** | [Hello GPT-4o](https://openai.com/index/hello-gpt-4o/) |

### 2.4 Google Gemini Live

| 项目 | 详情 |
|------|------|
| **最新** | Gemini 2.5 Flash Native Audio |
| **能力** | 30 种 HD 语音，24 种语言；可调语速；口音/角色语音；实时语音翻译 (Google Translate beta)；自动语言检测 |
| **技术** | 更精准的函数调用；robust 指令遵循；改进的对话流 |
| **链接** | [Gemini 音频更新](https://blog.google/products/gemini/gemini-audio-model-updates/), [Live 音频](https://blog.google/products/gemini/gemini-live-audio-updates/) |

### 2.5 ElevenLabs Conversational AI 2.0

| 项目 | 详情 |
|------|------|
| **产品** | Conversational AI 2.0 |
| **特色** | 自然轮换分析 (实时分析对话线索)；对话中自动语言切换；内置 RAG 知识检索；情感识别和自适应表达 |
| **个人助手** | 11.ai 个人语音助手产品 |
| **链接** | [Conversational AI 2.0](https://elevenlabs.io/blog/conversational-ai-2-0) |

---

## 三、视频原生音频 ★★★

> 2025 年下半年的关键突破：视频生成从"无声电影"进入"有声电影"时代。

### 3.1 里程碑时间线

```
2024.12  Veo 2 发布 → 4K 高质量，但无音频
2025.05  Veo 3 发布 → 首次原生音频同步 (对话 + 音效 + 环境音)
2025.09  Sora 2 发布 → 文本/图像生成视频 + 完整音频 (25 秒)
2025.10  Veo 3.1    → 4K + 参考图像 + 原生音频
2025.12  Kling 2.6  → 同步视听生成里程碑
2026.02  Kling 3.0  → 原生 4K 60fps + 多语言音频
```

### 3.2 各产品音频能力对比

| 产品 | 公司 | 发布 | 音频类型 | 视频时长 | 分辨率 |
|------|------|------|---------|---------|--------|
| **Sora 2** | OpenAI | 2025.09 | 对话 + 音效 + 背景音乐 + 环境音 | 25 秒 | - |
| **Veo 3** | Google | 2025.05 | 对话 + 音效 + 环境音 (同步) | - | 最高 4K |
| **Veo 3.1** | Google | 2025.10 | 帧间过渡音频 + 原生音频 | 8 秒 | 720p/1080p/4K |
| **Kling 2.6** | 快手 | 2025.12 | 语音 + 对话 + 旁白 + 唱歌 + rap + 环境音效 | - | - |
| **Kling 3.0** | 快手 | 2026.02 | 多语言/方言/口音原生音频 | 15 秒 | 原生 4K 60fps |

### 3.3 Sora 2 音频详解

| 项目 | 详情 |
|------|------|
| **发布** | 2025 年 9 月 30 日 |
| **音频生成** | 从文本提示或图像参考，一次生成中同步产生对话、音效、音乐和背景音 |
| **视频规格** | 最长 25 秒；改进的物理准确性和画面清晰度 |
| **平台** | sora.com, iOS 独立 App, API (计划中) |
| **链接** | [Sora 2 公告](https://openai.com/index/sora-2/), [System Card](https://openai.com/index/sora-2-system-card/) |

### 3.4 Kling 3.0 音频详解

| 项目 | 详情 |
|------|------|
| **发布** | 2026 年 2 月 5 日 |
| **产品线** | Video 3.0, Video 3.0 Omni, Image 3.0, Image 3.0 Omni |
| **音频** | 多语言、多方言、多口音原生音频；语音节奏与视觉动作同步 |
| **视频** | 首个原生 4K 60fps AI 视频；最长 15 秒；多镜头故事板 |
| **全模态** | 全模态输入/输出 (文本、图像、音频、视频) |
| **链接** | [Kling 3.0 公告](https://ir.kuaishou.com/news-releases/news-release-details/kling-ai-launches-30-model-ushering-era-where-everyone-can-be) |

---

## 四、技术深度：音频编解码器

> 现代语音/音频 AI 的基础组件。

```
音频编解码器的作用：

原始音频 (16-48kHz 采样)
   │
   ▼ 编码
┌──────────────────┐
│ 音频编解码器      │  离散化音频信号
│ (Audio Codec)     │  压缩比 100-300x
└────────┬─────────┘
         │ 离散 Token 序列
         │ (类似文本 Token)
         ▼
┌──────────────────┐
│ 语言模型          │  像处理文本一样
│ 或扩散模型        │  处理音频 Token
└────────┬─────────┘
         │ 生成的 Token
         ▼ 解码
      合成音频

主要编解码器：
  - EnCodec (Meta, 2022) → VALL-E 使用
  - SoundStream (Google) → AudioLM 使用
  - DAC (Descript Audio Codec) → Dia 使用
  - Mimi (Kyutai) → Sesame CSM 使用
  - FSQ (Finite Scalar Quantization) → CosyVoice 2 使用
```

---

## 五、关键时间线

| 时间 | 事件 |
|------|------|
| 2024.05 | GPT-4o 发布原生语音能力；ChatTTS 开源 |
| 2024.07 | SenseVoice 论文；Qwen2-Audio 论文 |
| 2024.10 | Whisper Large-v3 Turbo；F5-TTS 论文 |
| 2024.11 | Suno v4 |
| 2024.12 | CosyVoice 2 论文；Veo 2 (无音频) |
| 2025.02 | IndexTTS 论文 |
| 2025.03 | OpenAI gpt-4o-mini-tts + transcribe；Sesame CSM-1B；F5-TTS v1 |
| 2025.04 | Nari Labs Dia 1.6B |
| 2025.05 | Suno v4.5；Veo 3 (原生音频)；ACE-Step v1；Stable Audio Open Small |
| 2025.08 | OpenAI Realtime API GA |
| 2025.09 | Sora 2 (原生音频) |
| 2025.10 | Veo 3.1 |
| 2025.12 | Kling 2.6 (视听同步)；Gemini 2.5 Flash Native Audio |
| 2026.02 | Kling 3.0 (4K 60fps + 多语言音频) |

---

## 六、关键论文与资源

| 论文/项目 | 看点 | 链接 |
|-----------|------|------|
| VALL-E | 开创 TTS 语言模型范式 | [arXiv:2301.02111](https://arxiv.org/abs/2301.02111) |
| ACE-Step | 开源扩散式音乐生成 | [arXiv:2506.00045](https://arxiv.org/abs/2506.00045) |
| OpenAI Realtime API | 语音 Agent 基础设施 | [文档](https://platform.openai.com/docs/guides/realtime) |
| Sora 2 | 视频 + 原生音频 | [公告](https://openai.com/index/sora-2/) |
| Kling 3.0 | 4K 60fps + 多语言音频 | [公告](https://ir.kuaishou.com/news-releases/news-release-details/kling-ai-launches-30-model-ushering-era-where-everyone-can-be) |
