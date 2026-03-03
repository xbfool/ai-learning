# 本地 AI 体验项目

> 在自己家电脑上跑 LLM、图像、语音、视频生成。亲手触摸 AI，建立体感。

---

## 硬件参考

| 你的配置 | 能跑什么 |
|---------|---------|
| CPU 16GB RAM | Ollama 7B 量化、Whisper、ChatTTS、SenseVoice |
| RTX 3060 12GB | 上面全部 + SDXL/FLUX 图像生成、F5-TTS、LoRA 微调 7B、Wan 1.3B 视频 |
| RTX 4060 8-16GB | 上面全部 + 更快的推理 |
| RTX 4090 24GB | 上面全部 + GRPO 训练、14B 全精度、更长视频 |

---

## 项目一：本地运行 LLM（入门必做）

### 1.1 Ollama（10 分钟上手）

**安装**：从 https://ollama.com/download/windows 下载安装，双击运行。

**推荐模型**：

| 模型 | 命令 | 大小 | 特点 |
|------|------|------|------|
| **Qwen3-8B** | `ollama run qwen3:8b` | ~5GB | 中文最优，支持思考模式 |
| **DeepSeek-R1 7B** | `ollama run deepseek-r1:7b` | ~5GB | 推理模型，看思维链 |
| **Llama 3.1 8B** | `ollama run llama3.1:8b` | ~5GB | 英文强，Meta 出品 |
| **Phi-3 Mini** | `ollama run phi3` | ~2GB | 极小极快，入门首选 |

**性能预期**：RTX 3060 跑 7-8B Q4_K_M 模型，约 40+ tokens/s。

**进阶玩法**：
```bash
# Ollama 提供 OpenAI 兼容 API，可以给你的 Agent 项目用
curl http://localhost:11434/v1/chat/completions -d '{
  "model": "qwen3:8b",
  "messages": [{"role": "user", "content": "你好"}]
}'
```

### 1.2 LM Studio（图形界面）

**安装**：https://lmstudio.ai/

一键搜索下载模型，图形化对话界面，对新手极其友好。还支持开启本地 API 服务器和 MCP 服务器。

### 1.3 llama.cpp 手动量化（选做，了解原理）

```bash
# 克隆编译
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp && cmake -B build -DGGML_CUDA=ON && cmake --build build --config Release

# 量化（推荐 Q4_K_M）
./build/bin/llama-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M
```

**量化级别参考**（以 8B 模型为例）：

| 量化 | 大小 | 质量 | 推荐 |
|------|------|------|------|
| FP16 | ~16GB | 无损 | 不适合 12GB VRAM |
| Q8_0 | ~8GB | 极小损失 | 12GB VRAM 刚好 |
| **Q4_K_M** | **~5GB** | **小** | **日常推荐** |
| Q3_K_M | ~3.5GB | 中等 | 极限压缩 |

---

## 项目二：本地图像生成

### 2.1 ComfyUI + SDXL（推荐入门）

**安装**：
```bash
# 方法一：便携版（推荐新手）
# 从 GitHub Releases 下载 ComfyUI_windows_portable.zip
# 解压后运行 run_nvidia_gpu.bat

# 8GB VRAM 启动参数
python main.py --lowvram
```

**8-12GB VRAM 推荐模型**：

| 模型 | VRAM | 速度 | 推荐度 |
|------|------|------|--------|
| **SDXL** | 6-8GB | 20-40s | ★★★★★ 入门首选 |
| **SD 3.5 Medium** | 6-8GB | 15-30s | ★★★★ |
| **FLUX.1 Dev (Nunchaku 优化)** | 8GB | 15-30s | ★★★★ 需要安装 Nunchaku 节点 |
| FLUX.1 Schnell FP8 | 8GB | 30-60s | ★★★ |

**必装插件**：ComfyUI Manager（一键安装其他插件）
```bash
cd ComfyUI/custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Manager
```

### 2.2 进阶：ControlNet + IP-Adapter

| 工具 | 用途 |
|------|------|
| ControlNet Canny | 保持线稿结构生成 |
| ControlNet OpenPose | 控制人物姿态 |
| ControlNet Depth | 保持空间关系 |
| IP-Adapter | 用参考图控制风格（像"一张图的 LoRA"） |

---

## 项目三：本地语音 AI

### 3.1 Whisper 语音转文字（最简单）

```bash
pip install faster-whisper
```

```python
from faster_whisper import WhisperModel

model = WhisperModel("large-v3-turbo", device="cuda")  # 或 device="cpu"
segments, info = model.transcribe("audio.mp3")

for segment in segments:
    print(f"[{segment.start:.2f}s → {segment.end:.2f}s] {segment.text}")
```

| 模型 | VRAM | 速度 | 推荐 |
|------|------|------|------|
| tiny | ~1GB | 极快 | 测试用 |
| small | ~2GB | 快 | CPU 推荐 |
| **large-v3-turbo** | **~6GB** | **中等** | **GPU 推荐** |

### 3.2 SenseVoice 多功能识别（比 Whisper 快 17 倍）

```bash
pip install funasr
```

```python
from funasr import AutoModel
model = AutoModel(model="iic/SenseVoiceSmall", device="cuda:0")
result = model.generate(input="audio.wav")
# 输出：文字 + 语言 + 情感 + 音频事件（笑声/掌声等）
```

CPU 也能跑，速度比 Whisper 快很多。

### 3.3 ChatTTS 中文语音合成（4GB VRAM）

```bash
git clone https://github.com/jianchang512/ChatTTS-ui
cd ChatTTS-ui && pip install -r requirements.txt
python app.py
# 打开浏览器 http://localhost:9966
```

支持笑声、停顿等自然语音特征，4GB VRAM 就能跑。

### 3.4 F5-TTS 零样本语音克隆（6GB+ VRAM）

```bash
pip install f5-tts
f5-tts_infer-gradio  # 启动 WebUI
```

上传一段语音样本（几秒即可），生成该声音说任意文本。

### 3.5 CosyVoice 2 多语言 TTS

```bash
conda create -n cosyvoice python=3.10 && conda activate cosyvoice
git clone --recursive https://github.com/FunAudioLLM/CosyVoice.git
cd CosyVoice && pip install -r requirements.txt
```

零样本语音克隆 + 流式合成 + 多语言。

---

## 项目四：本地微调（进阶）

### Unsloth LoRA/QLoRA 微调

> RTX 3060 12GB 可以用 QLoRA 4-bit 微调 7B 模型，VRAM 仅需约 5-8GB。

**安装**：
```bash
conda create -n unsloth python=3.11 && conda activate unsloth
pip install torch --index-url https://download.pytorch.org/whl/cu121
pip install unsloth
```

**核心代码**：
```python
from unsloth import FastLanguageModel

# 加载 4-bit 预量化模型
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Qwen2.5-7B-bnb-4bit",
    max_seq_length=2048,
    load_in_4bit=True,
)

# 添加 LoRA 适配器
model = FastLanguageModel.get_peft_model(
    model, r=16, lora_alpha=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                     "gate_proj", "up_proj", "down_proj"],
)

# 训练...（500-5000 条数据即可见效）
```

**推荐配置**：

| 项目 | 推荐值 |
|------|--------|
| 基础模型 | Qwen2.5-7B / DeepSeek-R1-Distill-7B |
| 量化 | QLoRA 4-bit |
| LoRA Rank | 16 |
| 数据量 | 500-5000 条 |
| VRAM | ~5-8GB |

**部署微调结果**：导出为 GGUF → 在 Ollama 中使用。

**教程**：
- [Unsloth 官方文档](https://docs.unsloth.ai/)
- [Unsloth GitHub](https://github.com/unslothai/unsloth)

---

## 项目五：本地视频生成

### Wan 2.1 T2V（RTX 3060 可跑）

**推荐工具**：[Wan2GP](https://github.com/deepbeepmeep/Wan2GP)（专为低显存设计）

| 模型 | VRAM | 分辨率 | 时长 |
|------|------|--------|------|
| **Wan 2.1 T2V 1.3B** | **~8GB** | **480p** | **~5s** |
| Wan 2.1 T2V 14B GGUF | 12GB | 480p | ~5s |

```bash
git clone https://github.com/deepbeepmeep/Wan2GP.git
cd Wan2GP && pip install -r requirements.txt
python app.py  # 自动下载模型
```

**注意**：
- RTX 30xx 不支持 Sage Attention（已自动禁用）
- 14B 模型**必须用 GGUF 格式**，否则 OOM
- 生成速度较慢（几分钟一个短视频）

---

## 学习时间线建议

```
Week 1:  Ollama + LM Studio       → 本地 LLM 对话体验
Week 2:  ComfyUI + SDXL           → 图像生成体验
Week 3:  Whisper + ChatTTS        → 语音转写和合成
Week 4:  Wan2GP                   → 视频生成体验
Week 5-6: Unsloth 微调            → 定制自己的模型
Week 7+: 综合项目                 → 组合多种能力
```

---

## 关键链接汇总

| 类别 | 工具 | 链接 |
|------|------|------|
| LLM | Ollama | https://ollama.com/ |
| LLM | LM Studio | https://lmstudio.ai/ |
| LLM | llama.cpp | https://github.com/ggml-org/llama.cpp |
| 微调 | Unsloth | https://github.com/unslothai/unsloth |
| 图像 | ComfyUI | https://github.com/comfyanonymous/ComfyUI |
| 语音 | faster-whisper | https://github.com/SYSTRAN/faster-whisper |
| 语音 | SenseVoice | https://github.com/FunAudioLLM/SenseVoice |
| 语音 | ChatTTS UI | https://github.com/jianchang512/ChatTTS-ui |
| 语音 | F5-TTS | https://github.com/SWivid/F5-TTS |
| 语音 | CosyVoice | https://github.com/FunAudioLLM/CosyVoice |
| 视频 | Wan2GP | https://github.com/deepbeepmeep/Wan2GP |
