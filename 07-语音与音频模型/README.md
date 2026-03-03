# 07 - 语音与音频模型

> 2025-2026 年，语音与音频 AI 从辅助工具走向核心能力层。原生语音对话、视频音频同步生成、开源 TTS 全面开花。

## 总览

```
语音与音频模型
├── 语音合成 (TTS)          → 文字转语音，克隆、情感、多语言
│   ├── 商业闭源             → ElevenLabs, OpenAI TTS, Azure
│   └── 开源生态             → CosyVoice 2, F5-TTS, ChatTTS, Fish Speech
│
├── 语音识别 (ASR)          → 语音转文字，实时、多语言
│   ├── 通用 ASR             → Whisper v3, GPT-4o Transcribe, SenseVoice
│   └── 音频理解             → Qwen2-Audio (多模态音频理解)
│
├── 音乐与音频生成          → 文字/歌词生成音乐
│   ├── 商业产品             → Suno v4.5, Udio
│   └── 开源方案             → ACE-Step, Stable Audio Open
│
├── 语音 Agent              → 实时语音对话、语音助手
│   ├── 原生语音模型          → GPT-4o Voice, Gemini Live
│   └── 实时 API             → OpenAI Realtime API, ElevenLabs Conv AI
│
└── 视频原生音频             → 视频生成时同步生成音频
    ├── Sora 2               → 对话 + 音效 + 背景音
    ├── Veo 3 / 3.1          → 原生音频同步
    └── Kling 3.0            → 4K 60fps + 多语言音频
```

## 2025-2026 核心趋势

### 趋势 1：TTS 进入人类级质量

- VALL-E (Microsoft, 2023) 开创神经编解码器语言模型范式后，2025 年 TTS 全面突破
- Fish Speech OpenAudio S1 在 HuggingFace TTS-Arena-V2 Elo 排名第一
- CosyVoice 2 实现 150ms 首包延迟的流式合成
- F5-TTS 用 Flow Matching + DiT 实现全非自回归高质量合成
- 关键指标：MOS > 4.0，几乎无法与真人区分

### 趋势 2：原生语音对话取代 ASR + TTS 串联

- 传统：语音 → ASR → 文本 → LLM → 文本 → TTS → 语音（延迟 1-3 秒）
- 新范式：语音 → 原生语音模型 → 语音（延迟 232ms，GPT-4o）
- OpenAI Realtime API 于 2025 年 8 月 GA，成为语音 Agent 基础设施
- Google Gemini Live 支持 30 种 HD 语音、24 种语言

### 趋势 3：视频原生音频成为标配

- Sora 2 (2025.09)：首次实现视频 + 对话 + 音效 + 背景音同步生成
- Veo 3 (2025.05)：Google 的原生音频视频生成
- Kling 3.0 (2026.02)：4K 60fps + 多语言原生音频
- 标志着视频生成从"无声电影"进入"有声电影"时代

### 趋势 4：开源 TTS 生态爆发

- 2025 年至少 10+ 个高质量开源 TTS 模型发布
- CosyVoice 2 (Apache 2.0)、F5-TTS、ChatTTS、Fish Speech、Sesame CSM、Dia
- 开源质量已接近甚至超越部分商业方案

## 子文档

- [语音合成 (TTS)](./语音合成.md) - ElevenLabs、OpenAI TTS、CosyVoice 2、F5-TTS 等
- [语音识别与音频理解](./语音识别与音频理解.md) - Whisper、GPT-4o Transcribe、SenseVoice、Qwen2-Audio
- [音乐生成与语音Agent](./音乐生成与语音Agent.md) - Suno、ACE-Step、Realtime API、视频原生音频

## 核心参考

- [OpenAI Next-Gen Audio Models](https://openai.com/index/introducing-our-next-generation-audio-models/) - gpt-4o-mini-tts + transcribe 发布
- [CosyVoice 2 论文](https://arxiv.org/abs/2412.10117) - 流式/非流式统一 TTS 框架
- [F5-TTS 论文](https://arxiv.org/abs/2410.06885) - Flow Matching DiT TTS
- [Fish Audio OpenAudio S1](https://fish.audio/blog/introducing-s1/) - Dual-AR 架构 TTS
- [OpenAI Realtime API](https://openai.com/index/introducing-gpt-realtime/) - 语音 Agent 基础设施
