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

---

## 七、veRL 深度解析：HybridFlow 架构与 3D-HybridEngine

> **论文**：[HybridFlow: A Flexible and Efficient RLHF Framework](https://arxiv.org/abs/2409.19256) (EuroSys 2025)
> **代码**：[volcengine/verl](https://github.com/volcengine/verl)

### 7.1 单控制器 (Single-Controller) 的工作原理

veRL 的核心编程模型是 **混合控制器 (Hybrid-Controller)**，结合了「单控制器 (MPMD)」和「多控制器 (SPMD)」两种范式：

```
┌─────────────────────────────────────────────────────────┐
│              Single-Controller (PPORayTrainer)           │
│  ┌──────────────────────────────────────────────────┐   │
│  │ 全局编排层 (MPMD)                                 │   │
│  │  → 调度 rollout 生成                              │   │
│  │  → 触发 reward 计算                               │   │
│  │  → 分发分布式训练任务                              │   │
│  │  → 协调数据流在不同 Worker Group 之间的传递        │   │
│  └──────────────────────────────────────────────────┘   │
│         ↓ dispatch          ↑ collect                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │  Actor   │ │ Critic   │ │   Ref    │ │  Reward  │   │
│  │ Workers  │ │ Workers  │ │ Workers  │ │ Workers  │   │
│  │ (SPMD)   │ │ (SPMD)   │ │ (SPMD)   │ │ (SPMD)   │   │
│  │ FSDP/    │ │ FSDP/    │ │ FSDP     │ │ 独立     │   │
│  │ Megatron │ │ Megatron │ │          │ │ 计算     │   │
│  │ +vLLM    │ │          │ │          │ │          │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │
└─────────────────────────────────────────────────────────┘
```

**关键机制**：

1. **PPORayTrainer 作为中央控制器**：运行在单个进程上，负责加载完整的 prompt 批次，然后按照传输协议 (Transfer Protocol) 将数据分发到不同 Worker Group。

2. **数据分发与收集协议 (Dispatch/Collect Protocol)**：Worker 的方法通过 `@register` 装饰器显式定义数据的分割、分发和收集方式：
   - `Dispatch.DP_COMPUTE_PROTO`：将输入数据按 Data Parallel 维度切分，分发到各 Worker，收集输出后拼接
   - 控制流定义高层算子的执行顺序：先 Rollout → 再 Advantage Computation → 最后 Training Update

3. **v0.7 新增 TransferQueue**：解耦控制流和数据流，控制器仅发送指令和元数据，TransferQueue 负责实际数据传输（支持 PyTorch 张量的零拷贝和 RDMA）

**与多控制器对比**：

| 维度 | 单控制器 (veRL) | 多控制器 (OpenRLHF) |
|------|----------------|---------------------|
| 编排方式 | 中央进程统一调度 | 各模型独立控制器协调 |
| 调试难度 | 低（控制流集中） | 高（分布式状态难追踪） |
| 灵活性 | 高（Python 层面自由编排） | 受限于预定义通信模式 |
| 新算法扩展 | 几行代码改数据流 | 需修改多个控制器的协调逻辑 |

### 7.2 3D-HybridEngine 详解：Actor 模型的训练-推理切换

3D-HybridEngine 是 veRL 最核心的技术创新，解决了 RLHF 中 Actor 模型同时需要做训练和生成的独特挑战。

#### 问题背景

在 PPO/GRPO 等算法中，Actor 模型需要交替执行两种操作：
- **训练 (Training)**：需要 3D 并行（TP + PP + DP），使用 FSDP 或 Megatron-LM
- **生成 (Generation/Rollout)**：需要高效推理，使用 vLLM/SGLang 的推理优化

这两种操作对并行策略的要求截然不同。

#### Resharding 机制

```
训练阶段 (Training)                    生成阶段 (Generation)
┌────────────────────┐                ┌────────────────────┐
│ 并行策略: p-t-d     │                │ 并行策略: pg-tg-dg-d│
│ (PP=1, TP=2, DP=2) │  ── 切换 ──→  │ 重新排列并行组       │
│                     │                │                     │
│ GPU0: Shard_0       │                │ GPU0: 子集权重_A    │
│ GPU1: Shard_1       │                │ GPU1: 子集权重_B    │
│ GPU2: Shard_0'      │                │ GPU2: 子集权重_A'   │
│ GPU3: Shard_1'      │                │ GPU3: 子集权重_B'   │
└────────────────────┘                └────────────────────┘
         ↑                                      ↑
    FSDP/Megatron 分片                  vLLM 推理分片
```

**核心步骤**：

1. **并行组重排 (Parallel Group Rearrangement)**：
   - 训练时使用如 `(PP=1, TP=2, DP=2)` 的并行配置
   - 生成时切换为 `(PP_gen, TP_gen, DP_gen, DP)` 的不同配置
   - 通过战略性地重排生成阶段的并行组，使训练权重和生成权重在每个设备上产生**最大重叠**

2. **零冗余内存设计 (Zero Memory Redundancy)**：
   - 重排后，每个设备上的生成权重是训练权重的子集
   - 不需要额外的内存来存储生成阶段的模型副本
   - 训练权重可以直接在生成阶段被复用

3. **All-Gather 通信优化**：
   - 3D-HybridEngine 在模型并行组之间执行 All-Gather 操作来聚合参数
   - 然后每个设备根据自身所属的并行组，只保留所需的权重子集进行生成
   - **多个 All-Gather 操作并发执行**，减少通信开销

4. **完整切换流程**：
   ```
   训练完成 → 模型参数在 micro DP group 内更新
           → prompt 批次加载到各模型副本
           → 切换到生成并行配置，执行推理
           → 生成完成后，DP group 内 All-Gather 收集结果
           → 重新按 3D 并行策略分片，回到训练模式
   ```

### 7.3 分层 API 与 Auto-Mapping

HybridFlow 的三大组件：

1. **Hybrid Programming Model (分层 API)**：
   - 将 RLHF 复杂数据流中的计算依赖和数据依赖解耦并封装
   - 用户只需用几行 Python 代码定义 RL 数据流图
   - 支持 PPO、GRPO、DAPO 等算法的快速实现

2. **3D-HybridEngine**：如上所述

3. **Auto-Mapping Algorithm**：
   - 自动决定每个模型（Actor/Critic/Ref/Reward）在设备上的最优放置
   - 考虑内存约束、通信开销、计算负载均衡
   - 支持灵活的 colocated（共置）和 disaggregated（分离）部署模式

### 7.4 性能表现

- 与 SOTA 基线对比，吞吐量提升 **1.53x ~ 20.57x**
- v0.3.0 对比自身旧版提速约 **1.4x**
- 支持 FSDP、Megatron-LM、vLLM、SGLang、TensorRT-LLM 等多种后端

---

## 八、ROLL 深度解析：多角色架构与 Megatron-Core 集成

> **论文**：[ROLL: An Efficient and User-Friendly LLM RL Library](https://arxiv.org/abs/2506.06122)
> **ROLL Flash**：[Accelerating RLVR and Agentic Training with Asynchrony](https://arxiv.org/abs/2510.11345)
> **RollArt**：[Scaling Agentic RL Training via Disaggregated Infrastructure](https://arxiv.org/abs/2512.22560)
> **代码**：[alibaba/ROLL](https://github.com/alibaba/ROLL)

### 8.1 多角色分布式架构

ROLL 基于 Ray 构建了一套多角色分布式架构，将 RL 训练分解为三大角色：

```
┌──────────────────────────────────────────────────────────────┐
│                     Ray 调度层 (Master)                       │
│  AutoDeviceMapping: 自动设备映射                              │
│  支持 colocated (共置) 和 disaggregated (分离) 部署           │
├──────────────────┬────────────────────┬──────────────────────┤
│   Training Role  │   Rollout Role     │    Reward Role       │
│                  │                    │                      │
│  Megatron-Core   │   SGLang / vLLM    │   独立 GPU 集群      │
│  5D 并行策略:     │   Rollout Scheduler│   异步计算           │
│  TP + PP + DP    │   样本级生命周期管理 │                      │
│  + CP + EP       │                    │                      │
│                  │                    │                      │
│  DeepSpeed:      │   支持引擎:         │   支持:              │
│  ZeRO2/3/Offload │   - SGLang         │   - 基于规则的 RM    │
│                  │   - vLLM           │   - 神经网络 RM      │
│  FSDP2 + LoRA    │   - Megatron推理   │   - 可验证奖励       │
└──────────────────┴────────────────────┴──────────────────────┘
```

#### Megatron-Core 集成细节

ROLL 深度集成 Megatron-Core 作为训练后端，支持 **5D 并行策略**：

| 并行维度 | 全称 | 作用 |
|---------|------|------|
| **TP** | Tensor Parallelism | 将单个层的张量切分到多个 GPU |
| **PP** | Pipeline Parallelism | 将不同层分配到不同 GPU 流水线执行 |
| **DP** | Data Parallelism | 数据并行，每个 GPU 处理不同数据批次 |
| **CP** | Context Parallelism | 上下文并行，处理超长序列（动态上下文并行） |
| **EP** | Expert Parallelism | MoE 模型的专家并行 |

**对 MoE 模型的支持尤为关键**：Megatron-Core 提供灵活的 token 级别调度器，支持 token-dropping 和 token-dropless 两种 MoE 训练模式，跨全部 5 个并行维度协调 Attention 层和 MoE 层的不同并行方案。

**实战验证**：在阿里巴巴集群上使用 **3000+ GPU** 训练 **200B+ 参数的 MoE 模型**，持续运行约两周无中断，验证了 ROLL 的稳定性和容错能力。

#### SGLang/vLLM Rollout 集成

ROLL 为推理/生成阶段提供了灵活的引擎选择：

```python
# ROLL 的 Rollout 配置示例（概念性）
rollout_config:
  engine: "sglang"          # 或 "vllm"
  scheduler: "sample_level" # 样本级调度
  max_concurrent: 1024      # 并发样本数
  dynamic_batching: true    # 动态批处理
```

**Rollout Scheduler（推理调度器）**的核心特点：
- **样本级生命周期管理**：用户可以控制每个样本的生命周期——决定何时、在哪里执行每个样本的每个阶段
- **与传统批级调度的对比**：传统方法以批次为粒度调度，一个批次必须等所有样本完成；ROLL 在单个样本粒度上调度，一个样本完成就可以立即开始下一步
- **管线化重叠**：一个样本的 LLM 生成、另一个样本的环境交互、第三个样本的奖励计算可以同时进行

### 8.2 ROLL Flash：GPU 部分重叠技术

ROLL Flash 是 ROLL 的异步加速扩展，建立在两个核心设计原则上：**细粒度并行 (Fine-Grained Parallelism)** 和 **Rollout-Train 解耦**。

#### 异步训练架构

```
同步模式（传统）：
  Time  ──────────────────────────────────→
  GPU:  [Rollout████████][Train████][Rollout████████][Train████]
        ←── 推理 GPU 忙 ──→← 推理空闲 →←── 推理 GPU 忙 ──→

ROLL Flash 异步模式：
  Time  ──────────────────────────────────→
  Rollout GPU: [Gen_1][Gen_2][Gen_3][Gen_4][Gen_5][Gen_6]...
  Train GPU:      [Train_1][Train_2][Train_3][Train_4]...
                     ↑ 无需等待所有 rollout 完成
```

#### GPU 部分重叠 (Partial Overlapping) 技术

ROLL Flash 引入了四个关键组件来实现高效的异步训练：

1. **LLMProxy**：LLM 推理的代理层，管理模型权重的版本和更新
2. **EnvManager**：环境管理器，处理 Agent 与环境的多轮交互
3. **SampleBuffer**：样本缓冲区，暂存已完成 rollout 但尚未用于训练的样本
4. **AsyncController**：异步控制器，协调 rollout 和 training 的节奏

**细粒度并行的实现**：

```
样本级流水线 (Sample-Level Pipeline):

Sample_1: [LLM生成] → [环境交互] → [奖励计算] → 进入 Buffer
Sample_2:    [LLM生成] → [环境交互] → [奖励计算] → 进入 Buffer
Sample_3:       [LLM生成] → [环境交互] → [奖励计算] → ...
                ↑ 样本间重叠，消除等待

Agentic 场景的多轮重叠：
Sample_1: [Gen][Env][Gen][Env][Gen][Env] → Reward
Sample_2:   [Gen][Env][Gen][Env][Gen] → ...
Sample_3:      [Gen][Env][Gen] → ...
                ↑ 环境交互延迟被完全隐藏
```

#### 异步比率 (Asynchronous Ratio) α

ROLL Flash 引入 **异步比率 α** 来控制数据陈旧性：

- **定义**：α 是每个样本允许的最大策略版本差——即当前训练策略版本与生成该样本时使用的策略版本之间的差距上限
- **作用**：防止过时的 rollout 数据降低训练质量
- **调节**：α = 0 退化为完全同步，α → ∞ 为完全异步，实际使用中 α 取 1~4 效果最佳

#### 性能成果

| 任务类型 | 加速比 | 说明 |
|---------|-------|------|
| RLVR (可验证奖励) | **2.24x** | 数学推理等有确定答案的任务 |
| Agentic (多步交互) | **2.72x** | Agent 与环境多轮交互的训练 |

### 8.3 RollArt：Agentic RL 的分离式基础设施

ROLL 系列的最新进展是 **RollArt** ([arXiv:2512.22560](https://arxiv.org/abs/2512.22560))，专门针对 Agentic RL 训练的异构计算需求：

- **问题**：Agentic RL 的 workload 高度异构——计算密集的 Prefill 阶段、带宽密集的 Decode 阶段、CPU 密集的环境模拟
- **方案**：分离式基础设施 (Disaggregated Infrastructure)，将不同计算阶段放在最适合的硬件上
- **挑战**：分离式部署引入了大量同步开销和资源利用率下降

### 8.4 三类用户的使用路径

```
Tech Pioneer (追求极致性能):
  → Megatron-Core + 5D 并行 + ROLL Flash 异步
  → 适合 200B+ MoE 模型的大规模训练

Developer (灵活控制):
  → FSDP2 + LoRA + 自定义 Rollout Scheduler
  → 灵活控制训练工作流的每个环节

Researcher (快速迭代):
  → 简单配置 + 开箱即用的算法实现
  → 快速在不同算法和模型之间切换实验
```

---

## 九、AReaL 深度解析：全异步设计与 boba² 方法

> **论文**：[AReaL: A Large-Scale Asynchronous Reinforcement Learning System for Language Reasoning](https://arxiv.org/abs/2505.24298)
> **代码**：[inclusionAI/AReaL](https://github.com/inclusionAI/AReaL)
> **模型**：[AReaL-boba-2 系列](https://huggingface.co/inclusionAI/AReaL-boba-2-32B)

### 9.1 全异步设计的核心思想

#### 同步 vs 异步：根本性的效率差距

```
同步 RL 训练的效率瓶颈：
┌──────────────────────────────────────────────────────┐
│ Rollout Phase:                                        │
│ Sample_1: [====]                                      │
│ Sample_2: [========]                                  │
│ Sample_3: [==============]  ← 长尾样本               │
│ Sample_4: [====]                                      │
│           ↑ 必须等所有样本完成才能开始训练              │
│                              ↓                        │
│ Training: .................. [████████████]            │
│           ← 训练 GPU 空闲 →  ← 推理 GPU 空闲 →       │
└──────────────────────────────────────────────────────┘

AReaL 全异步训练：
┌──────────────────────────────────────────────────────┐
│ Rollout Workers (持续运行，互不等待):                   │
│ Worker_1: [Gen][Gen][Gen][Gen][Gen][Gen]...           │
│ Worker_2: [Gen][Gen][Gen][Gen][Gen]...                │
│ Worker_3: [Gen][Gen][Gen][Gen]...                     │
│                                                       │
│ Training Workers (数据就绪即训练):                      │
│ Trainer:  [Train][Train][Train][Train][Train]...      │
│           ↑ 无需等待所有 rollout 完成                   │
│           ↑ 每凑齐一个 batch 就开始训练                 │
└──────────────────────────────────────────────────────┘
→ 所有 GPU 几乎 100% 利用率
```

#### 系统架构

AReaL 的系统由 4 个核心组件构成：

```
┌─────────────────────────────────────────────────────────┐
│                    Master Controller                     │
│  - 协调 Rollout 和 Training 的节奏                       │
│  - 管理模型版本                                          │
│  - 分发 prompt 和收集完成的样本                           │
├────────────────────────┬────────────────────────────────┤
│   Rollout Workers      │     Training Workers           │
│                        │                                │
│ ┌──────────────────┐   │   ┌──────────────────────┐     │
│ │ Interruptible    │   │   │ 标准分布式训练         │     │
│ │ Rollout Worker   │   │   │ (FSDP/自研后端)       │     │
│ │                  │   │   │                      │     │
│ │ 两种请求:        │   │   │ 接收: 来自 buffer     │     │
│ │ 1. generate()    │   │   │ 的样本批次            │     │
│ │    生成回复       │   │   │                      │     │
│ │ 2. update_weights│   │   │ 输出: 更新后的模型    │     │
│ │    中断并加载新权重│   │   │ 权重 → 同步回        │     │
│ │                  │   │   │ Rollout Workers       │     │
│ └──────────────────┘   │   └──────────────────────┘     │
└────────────────────────┴────────────────────────────────┘
```

**关键设计点**：

1. **可中断的 Rollout Worker**：支持两种请求——`generate()` 用于生成回复，`update_weights()` 会中断所有正在进行的生成并加载新版本参数
2. **流式生成 (Streaming Generation)**：每个 Rollout Worker 持续生成新输出，不等待其他 Worker
3. **版本管理**：Master 追踪每个样本由哪个版本的模型生成，用于后续的 staleness 处理

### 9.2 boba² 方法：解决数据陈旧性的算法创新

AReaL 最核心的算法贡献是 **boba²（double-boba）** 方法，通过「算法-系统协同设计 (Algorithm-System Co-Design)」来保证全异步训练的稳定性。

#### 数据陈旧性 (Data Staleness) 的问题

在全异步模式下，一个训练 batch 中的样本可能来自多个不同版本的策略模型：

```
训练 Batch 示例:
┌─────────────────────────────────────┐
│ Sample_A: 由 π_θ(v=10) 生成        │  ← 当前版本
│ Sample_B: 由 π_θ(v=8) 生成         │  ← 2 步前
│ Sample_C: 由 π_θ(v=6) 生成         │  ← 4 步前
│ Sample_D: 由 π_θ(v=9) 生成         │  ← 1 步前
└─────────────────────────────────────┘
当前训练策略版本: v=10

问题: Sample_C 是 4 个版本前的旧数据
     → 行为策略 π_old 与当前策略 π_θ 分布差距大
     → 直接使用标准 PPO 会导致训练不稳定
```

#### Decoupled PPO：解耦行为策略与近端策略

AReaL 采用 **Decoupled PPO** 来处理陈旧数据，核心思想是将标准 PPO 中的两个角色解耦：

**标准 PPO 的问题**：
- 标准 PPO 的重要性权重 (importance weight) 和信任域约束 (trust region) 都依赖同一个「行为策略」π_old
- 异步训练中，π_old 可能是很旧的版本，导致重要性权重偏差大

**Decoupled PPO 的解决方案**：
- **行为策略 (Behavior Policy) π_b**：实际生成样本时使用的（可能是旧的）策略，用于计算重要性采样权重，实现离策略修正 (Off-Policy Correction)
- **近端策略 (Proximal Policy) π_prox**：一个「锚点」策略，位于行为策略和当前策略之间，用于定义信任域，约束更新幅度

```
策略空间示意:

π_b (旧)        π_prox (中间锚点)        π_θ (当前)
  ·─────────────────·──────────────────────·
  行为策略           近端策略               训练目标
  (生成样本用)       (信任域锚点)           (正在优化)

关键 insight: π_prox 不需要通过网络前向计算，
只需在对数概率空间中，对 π_b 和 π_θ 做插值即可
```

#### 陈旧性处理的多层防线

AReaL 采用多层策略确保异步训练的稳定性：

1. **版本差过滤**：超过陈旧性阈值（如 8 步）的样本直接丢弃
2. **解耦 PPO 目标**：通过近端策略解耦 off-policy 修正和信任域约束
3. **自适应 Clipping**：根据数据陈旧程度调整 PPO 的 clip 范围
4. **KL/陈旧性正则化**：添加与版本差成正比的正则项，陈旧数据的更新幅度更保守
5. **工作负载均衡**：从系统层面平衡 Rollout 和 Training 的速度，尽量控制版本差不会太大

**实验验证**：使用旧到 8 步前的样本进行训练，模型性能不会下降，这意味着系统可以容忍较大的异步度而不牺牲精度。

### 9.3 2.77x 加速的实现原理

AReaL 的加速来自多个层面：

| 加速来源 | 贡献 | 说明 |
|---------|------|------|
| 消除同步等待 | 主要 | Rollout 和 Training 完全并行 |
| 消除长尾等待 | 重要 | 不需要等最慢的样本完成 |
| GPU 利用率提升 | 重要 | 从 ~40-60% 提升到 ~90%+ |
| 流式生成 | 辅助 | Rollout Worker 持续产出样本 |

**量化结果**：
- **吞吐量**：相比 SOTA 同步系统提升 **2.57x**
- **端到端训练速度**：提升 **2.77x**
- **线性扩展**：在最多 **512 GPU** 上保持线性扩展效率
- **对比对象**：DeepScaleR、DeepCoder 等同步系统

### 9.4 ASearcher：端到端训练的搜索 Agent

AReaL 团队使用该框架训练了 **ASearcher**，一个 SOTA 级别的搜索推理 Agent：

- **ASearcher-Web-QwQ-v2 (32B)**：在开源 Agent 中达到最高水平
  - GAIA: Avg@4 = 58.7
  - xBench-DeepSearch: Avg@4 = 51.1
  - Frames: Avg@4 = 74.5
- **boba² 模型系列**：基于 Qwen3 通过异步 RL 训练
  - 在 LiveCodeBench、Codeforces、CodeContests 等编程基准上达到 SOTA
  - 提供 8B、14B、32B 等多个规模

### 9.5 AReaL vs AReaL-lite

| 维度 | AReaL (完整版) | AReaL-lite (轻量版) |
|------|---------------|-------------------|
| 目标 | 极致训练效率 | 研究者友好 |
| 异步 | 全异步 | 支持但非必须 |
| 设计重心 | 系统优化 | API 设计 |
| 扩展性 | 512+ GPU | 小规模集群 |
| 适合 | 生产环境、大规模训练 | 快速原型、算法研究 |

---

## 十、实战对比：什么场景用哪个框架

### 场景 1：学术研究者，8 张 A100，想复现 GRPO 论文

**推荐：veRL**

```
理由：
- 单控制器架构，调试方便
- 支持的算法最多（GRPO, PPO, DAPO, REINFORCE++ 等）
- 分层 API 可以快速修改算法细节
- 文档和社区生态成熟

示例配置：
  模型: Qwen2.5-7B
  算法: GRPO
  训练后端: FSDP (8 GPU)
  推理后端: vLLM
  预计设置时间: 半天
```

### 场景 2：大厂团队，1000+ GPU，训练 200B MoE 的生产级 RL

**推荐：ROLL**

```
理由：
- Megatron-Core 深度集成，5D 并行策略适合超大规模 MoE
- 已在阿里 3000+ GPU 集群验证过 200B MoE 连续两周训练
- 强大的容错和 checkpoint 机制
- AutoDeviceMapping 自动化设备分配

示例配置：
  模型: 200B MoE (TP=8, PP=4, EP=8, CP=2, DP=4)
  算法: GRPO
  训练后端: Megatron-Core
  推理后端: SGLang
  部署模式: disaggregated (训练和推理分开集群)
```

### 场景 3：GPU 预算有限，需要最大化训练速度

**推荐：AReaL**

```
理由：
- 全异步设计最大化 GPU 利用率
- 同样的 GPU 预算下，训练速度快 2.77x
- boba² 方法保证异步训练不掉精度
- 线性扩展到 512 GPU

示例配置：
  模型: Qwen3-32B
  算法: Decoupled PPO (GRPO 变体)
  Rollout: 流式生成
  版本容忍度: 8 步
  预计对比同步: 2.5-2.7x 加速
```

### 场景 4：训练 Agent（如搜索 Agent、代码 Agent）

**推荐：AReaL 或 ROLL Flash**

```
Agent 训练的特殊性：
- Rollout 时间极长（多步环境交互）
- 不同样本的 rollout 时间方差极大
- 同步等待的浪费最为严重

AReaL 优势：全异步，完全消除等待
ROLL Flash 优势：样本级调度 + 环境级异步并行

对比：
  同步基线:       [=== rollout 10min ===][train 2min] → GPU 利用率 ~17%
  AReaL 异步:     rollout 和 train 完全并行          → GPU 利用率 ~90%
  ROLL Flash:    样本级流水线 + 部分重叠              → GPU 利用率 ~80%
```

### 场景 5：初学者，想了解 RL 训练全流程

**推荐：OpenRLHF 或 TRL**

```
理由：
- OpenRLHF 代码结构清晰，社区活跃
- TRL 与 HuggingFace 生态无缝集成
- 文档丰富，教程完善
- 不需要理解复杂的分布式系统设计

之后进阶路线：
  TRL → OpenRLHF → veRL → ROLL/AReaL
```

### 综合决策树

```
开始
 │
 ├─ GPU 少于 16 张？
 │   ├─ 是 → 研究目的？
 │   │         ├─ 是 → veRL 或 AReaL-lite
 │   │         └─ 否 → TRL 或 OpenRLHF
 │   │
 │   └─ 否 → GPU 超过 256 张？
 │             ├─ 是 → MoE 模型？
 │             │         ├─ 是 → ROLL (Megatron-Core 5D 并行)
 │             │         └─ 否 → GPU 利用率优先？
 │             │                   ├─ 是 → AReaL
 │             │                   └─ 否 → veRL 或 ROLL
 │             │
 │             └─ 否 (16-256 GPU) → Agent 训练？
 │                                    ├─ 是 → AReaL 或 ROLL Flash
 │                                    └─ 否 → veRL (灵活性最高)
```

---

## 十一、关键论文与参考资料

| 框架 | 论文 | 会议/状态 |
|------|------|----------|
| veRL / HybridFlow | [arXiv:2409.19256](https://arxiv.org/abs/2409.19256) | EuroSys 2025 |
| ROLL | [arXiv:2506.06122](https://arxiv.org/abs/2506.06122) | 2025 |
| ROLL Flash | [arXiv:2510.11345](https://arxiv.org/abs/2510.11345) | 2025 |
| RollArt | [arXiv:2512.22560](https://arxiv.org/abs/2512.22560) | 2025 |
| AReaL | [arXiv:2505.24298](https://arxiv.org/abs/2505.24298) | 2025 |
| AReaL-Hex (异构 GPU) | [arXiv:2511.00796](https://arxiv.org/abs/2511.00796) | 2025 |
| A-3PO (陈旧性感知) | [arXiv:2512.06547](https://arxiv.org/abs/2512.06547) | 2025 |
| OpenRLHF | [GitHub](https://github.com/OpenRLHF/OpenRLHF) | 2023- |
| PPO 原始论文 | [arXiv:1707.06347](https://arxiv.org/abs/1707.06347) | 2017 |
