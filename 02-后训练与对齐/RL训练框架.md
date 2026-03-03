# RL 训练框架

> 大规模 RL 训练需要协调 Actor、Critic、Reference、Reward 多个模型在 GPU 集群上的运行。本文详解三大开源框架。

## 一、框架全景

```
RL 训练框架
├── veRL (字节跳动/火山引擎)    → 单控制器, HybridFlow ★
├── ROLL (阿里淘天)             → 多角色分布式, Megatron + SGLang ★
├── AReaL (蚂蚁集团)            → 全异步, 算法-系统协同设计 ★
├── OpenRLHF (社区)             → 多控制器, Ray
└── TRL (HuggingFace)           → 简单易用, 小规模
```

## 二、veRL (Volcano Engine RL)

- **开发者**：字节跳动 / 火山引擎
- **GitHub**：[volcengine/verl](https://github.com/volcengine/verl)
- **论文**：HybridFlow (EuroSys 2025)
- **定位**：灵活、高效、生产就绪的 RL 训练框架

### 核心设计

#### 单控制器架构 (Single-Controller)
```
┌───────────────────────────────┐
│        Controller (Ray)       │  ← 中央编排器
├───────┬───────┬───────┬──────┤
│ Actor │Critic │  Ref  │Reward│  ← Worker 角色
│Workers│Workers│Workers│Workers│
└───────┴───────┴───────┴──────┘

控制流程：
  Controller 指挥所有 Worker 的执行顺序
  → 控制流清晰，易于调试
  → 与 OpenRLHF 的多控制器形成对比
```

#### 3D-HybridEngine
- 训练时用 FSDP/Megatron 的分布式策略
- 推理 (Rollout) 时切换到 vLLM 的推理引擎
- **关键**：Actor 模型在训练和推理之间高效切换 (resharding)
- 消除内存冗余，减少通信开销

### 支持的算法
PPO, GRPO, GSPO, ReMax, REINFORCE++, RLOO, PRIME, DAPO, DrGRPO

### 关键成果
- **DAPO**：在 Qwen2.5-32B 上 AIME 2024 达到 50 分
- **Doubao-1.5-pro**：veRL 训练的 RL scaling preview 达到 O1 级别数学性能
- v0.3.0：对比旧版提速 ~1.4x

### 适合场景
- 中大规模 RL 训练
- 需要灵活切换算法
- 研究和生产双用途

---

## 三、ROLL (Reinforcement Learning Optimization for Large-scale Learning)

- **开发者**：阿里巴巴淘天集团
- **GitHub**：[alibaba/ROLL](https://github.com/alibaba/ROLL)
- **论文**：[arXiv:2506.06122](https://arxiv.org/abs/2506.06122)，后续有 ROLL Flash ([arXiv:2510.11345](https://arxiv.org/abs/2510.11345))
- **定位**：高效、可扩展、用户友好的大规模 RL 训练库

### 核心设计

#### 多角色分布式架构
```
┌──────────────────────────────────────┐
│            Ray 调度层                 │
├──────────┬────────────┬──────────────┤
│ Training │  Rollout   │   Reward     │
│ (Megatron│  (SGLang/  │  (独立      │
│  -Core)  │   vLLM)    │   计算)     │
└──────────┴────────────┴──────────────┘

特点：
- 训练引擎：Megatron-Core (高性能)
- 推理引擎：SGLang / vLLM (灵活选择)
- Ray 做灵活的资源分配和异构调度
```

#### ROLL Flash (Part II)
- 加速 RLVR (RL with Verifiable Rewards) 和 Agentic 训练
- GPU 部分重叠 (Partial Overlapping)：训练和推理计算交错
- 支持 FSDP2 + Megatron + LoRA 组合

### 三类目标用户
1. **Tech Pioneers**：追求大规模、低成本、容错训练
2. **Developers**：需要灵活控制训练工作流
3. **Researchers**：追求快速实验迭代

### 实际应用
- TaoSR-AGRL：淘宝搜索中的 LLM 相关性排序（服务亿级用户）
- Retrieval-GRPO：淘宝搜索的多目标检索优化

### 适合场景
- 超大规模训练（Megatron-Core 支撑）
- 生产环境的容错需求
- 电商/搜索等实际业务落地

---

## 四、AReaL (Asynchronous Reinforcement Learning)

- **开发者**：蚂蚁集团 + 清华 IIIS
- **GitHub**：[inclusionAI/AReaL](https://github.com/inclusionAI/AReaL)
- **论文**：[arXiv:2505.24298](https://arxiv.org/abs/2505.24298)
- **定位**：全异步 RL 训练，最大化 GPU 利用率

### 核心设计

#### 全异步训练 (Fully Asynchronous)
```
同步 RL 训练：
  Rollout 完所有样本 → 统一训练 → 再 Rollout → ...
  问题：Rollout 时训练 GPU 空闲，训练时推理 GPU 空闲

AReaL 全异步：
  Rollout 和 Training 同时进行，互不等待
  ┌────────────────────────────────────┐
  │ Rollout:  [===] [===] [===] [===]  │
  │ Training:   [===] [===] [===]      │
  │              ↑ 无需等待对齐         │
  └────────────────────────────────────┘
  → 最大化所有 GPU 的利用率
```

#### 算法-系统协同设计 (Algorithm-System Co-Design)
- 异步训练引入的 "数据陈旧性" 需要算法层面的修正
- boba² 方案：确保全异步下的训练稳定性

### 关键成果
- **2.77x 加速**：相比同步系统，不牺牲精度
- **ASearcher**：用 AReaL 端到端训练的 SOTA 搜索 Agent

### 版本
- **AReaL**：完整版，追求极致效率
- **AReaL-lite**：轻量版，面向研究者，API 设计优先于系统优化

### 适合场景
- GPU 资源昂贵，需要最大化利用率
- Agent 训练（多步交互，rollout 时间长）
- 研究快速原型（AReaL-lite）

---

## 五、框架对比

| 维度 | veRL | ROLL | AReaL | OpenRLHF |
|------|------|------|-------|----------|
| **架构** | 单控制器 (Ray) | 多角色分布式 (Ray) | 全异步 | 多控制器 (Ray) |
| **训练引擎** | FSDP + Megatron | Megatron-Core | 自研 | FSDP + DeepSpeed |
| **推理引擎** | vLLM | SGLang / vLLM | 自研 | vLLM |
| **核心优势** | 灵活 + 调试友好 | 大规模 + 生产就绪 | 极致效率 (2.77x) | 社区活跃 |
| **异步支持** | 部分 | 部分重叠 | 全异步 | 无 |
| **算法覆盖** | 最广 (10+) | GRPO, PPO 为主 | GRPO, PPO | PPO, GRPO, DPO, KTO |
| **适合用户** | 研究+生产 | 大厂生产 | 效率导向 | 入门+社区 |
| **开源时间** | 2024 | 2025 | 2025 | 2023 |

## 六、选择建议

```
"我是研究者，想快速实验不同 RL 算法" → veRL 或 AReaL-lite
"我需要在大规模集群上做生产级训练" → ROLL
"我的 GPU 资源有限，要最大化利用率" → AReaL
"我是初学者，先跑通一个 baseline" → OpenRLHF 或 TRL
"我要训练 Agent（多步交互）" → AReaL（异步优势大）或 ROLL Flash
```
