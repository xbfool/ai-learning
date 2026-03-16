# 云端 Agent 服务架构最佳实践

> 更新：2026-03-16
> 场景：Agent 跑在云端为用户提供服务（SaaS），不是用户本地运行。
> 这和 Claude Code 在本地跑 Agent Teams 是完全不同的架构挑战。

---

## 一、本地 vs 云端：根本差异

| 维度 | 本地 Agent (Claude Code) | 云端 Agent 服务 |
|------|------------------------|----------------|
| 用户数 | 1 个开发者 | 成百上千并发用户 |
| 状态 | 进程内，天然隔离 | 必须外部化，多实例共享 |
| 成本 | 用户自己的 token | 你的钱包，token 用量 = 运营成本 |
| 失败影响 | 自己重试 | 用户体验受损，可能丢数据 |
| 扩缩容 | 不需要 | 核心挑战 |
| 安全 | 信任本地环境 | Zero Trust，多租户隔离 |

**核心区别**：本地模式的奢侈（独占上下文、无限重试、不计成本）在云端全部变成约束。

---

## 二、Anthropic 官方的六大模式选型

Anthropic 在 《Building Effective Agents》中定义了从简到繁的模式梯度：

```
复杂度 ↑
│
│  ┌─────────────────────────┐
│  │ 6. Autonomous Agent     │  ← 开放式任务，步骤不可预测
│  ├─────────────────────────┤
│  │ 5. Orchestrator-Workers │  ← 动态子任务分解 + 并行
│  ├─────────────────────────┤
│  │ 4. Evaluator-Optimizer  │  ← 生成 + 评估迭代优化
│  ├─────────────────────────┤
│  │ 3. Parallelization      │  ← 同一任务多路并行或投票
│  ├─────────────────────────┤
│  │ 2. Routing              │  ← 分类输入，走不同处理路径
│  ├─────────────────────────┤
│  │ 1. Prompt Chaining      │  ← 固定步骤串联
│  └─────────────────────────┘
│
复杂度 ↓
```

**Anthropic 的核心建议**：

> "Start simple. Begin by using LLM APIs directly — many patterns can be implemented in a few lines of code."
> 从最简单的开始，只在复杂度确实需要时才升级模式。

### 云端服务的选型指南

| 你的场景 | 推荐模式 | 理由 |
|---------|----------|------|
| 确定性流程（每次步骤相同） | Prompt Chaining | 可预测、成本低、延迟稳定 |
| 不同类型输入需要不同处理 | Routing | 按类型走专门路径，每条路径可独立优化 |
| 需要多角度分析同一输入 | Parallelization | 并行不增加延迟，投票提高质量 |
| 复杂任务，子任务不可预测 | Orchestrator-Workers | 灵活但成本高，需要控制 |
| 需要质量保证的生成任务 | Evaluator-Optimizer | 迭代提升质量，但增加延迟和成本 |
| 完全开放式任务 | Agent | 最灵活也最贵，必须有安全边界 |

---

## 三、云端 Agent 架构的五大核心挑战

### 挑战 1: 状态管理 — Session 不能放内存里

本地 Agent 的状态在进程里，云端不行——实例可能随时销毁或扩容。

```
❌ 错误做法：
Agent 实例 A          Agent 实例 B
┌──────────┐          ┌──────────┐
│ session 1 │         │          │
│ session 2 │         │          │  ← 实例 B 不知道任何会话
│ session 3 │         │          │
└──────────┘          └──────────┘
  ↑ 如果实例 A 挂了，session 全丢

✅ 正确做法：
Agent 实例 A          Agent 实例 B
┌──────────┐          ┌──────────┐
│ stateless │         │ stateless │  ← 任何实例可以接任何请求
└────┬─────┘          └────┬─────┘
     │                     │
     └──────────┬──────────┘
                ▼
    ┌───────────────────┐
    │  外部状态存储       │
    │  Redis / DynamoDB  │
    │  ┌─────────────┐  │
    │  │ session 1   │  │
    │  │ session 2   │  │
    │  │ session 3   │  │
    │  └─────────────┘  │
    └───────────────────┘
```

**三层记忆架构**：

| 层 | 存什么 | 存哪里 | 访问延迟 |
|----|--------|--------|----------|
| 短期记忆 | 当前对话上下文、工作状态 | Redis (内存) | < 1ms |
| 情景记忆 | 特定交互事件、审计日志 | PostgreSQL / DynamoDB | < 10ms |
| 长期记忆 | 用户画像、知识库、历史模式 | 向量数据库 + 关系数据库 | < 100ms |

### 挑战 2: Token 成本控制 — 你的钱包在燃烧

Anthropic 的数据：
- 单 Agent 调用 ≈ 普通对话的 **4 倍** token
- 多 Agent 系统 ≈ 普通对话的 **15 倍** token

```
用户一次请求的成本链：

用户输入 (少量 token)
  → Orchestrator 推理 (数千 token)
    → Worker A 搜索 + 推理 (数千 token)
    → Worker B 分析 + 推理 (数千 token)
    → Worker C 生成 + 推理 (数千 token)
  → Orchestrator 汇总 (数千 token)
  → 最终回复 (少量 token)

真正输出给用户的可能只占 5%，95% 是"思考成本"
```

**控制策略**：

| 策略 | 实现方式 | 效果 |
|------|----------|------|
| 语义缓存 | 相似问题直接返回缓存结果 | 减少 40-69% API 调用 |
| 模型分层 | 简单任务用 Haiku，复杂任务才用 Opus | 成本降 5-10 倍 |
| Token 预算 | 每个 Agent 设硬上限，超限终止 | 防止失控消耗 |
| 增量处理 | 只传 delta 变化，不重传全量上下文 | 减少 40% 重读成本 |
| 动态路由 | 先判断复杂度，简单问题不走 Agent | 避免杀鸡用牛刀 |

### 挑战 3: 可靠性 — 5% 失败率在多步骤下是灾难

```
单步成功率 95%
  → 5 步链路成功率: 0.95^5 = 77%
  → 10 步链路成功率: 0.95^10 = 60%
  → 20 步链路成功率: 0.95^20 = 36%  ← 大概率失败！
```

**解法**：

```
┌──────────────────────────────────────────────────┐
│                 可靠性工程栈                       │
│                                                    │
│  1. 检查点 (Checkpoint)                            │
│     每一步执行完持久化状态，失败从最近检查点恢复        │
│                                                    │
│  2. 重试 + 指数退避                                 │
│     工具调用失败自动重试，间隔递增                     │
│                                                    │
│  3. 熔断器 (Circuit Breaker)                       │
│     某个工具连续失败 N 次，暂停调用，走降级路径         │
│                                                    │
│  4. 超时控制                                       │
│     每步设超时，整体任务也设超时                       │
│                                                    │
│  5. 降级策略                                       │
│     Agent 完全失败时，返回部分结果 + 说明             │
│                                                    │
│  6. Human-in-the-loop 门控                         │
│     高风险操作（>$50 / 影响范围大）需人工确认         │
│                                                    │
└──────────────────────────────────────────────────┘
```

### 挑战 4: 多租户隔离 — 用户 A 不能看到用户 B 的数据

```
请求流：

用户 A ──→ ┌──────────┐     ┌──────────────────┐
           │ API GW   │────→│ Agent Runtime     │
用户 B ──→ │ 认证+限流 │     │                  │
           └──────────┘     │ ┌──────────────┐  │
                            │ │ Session 隔离  │  │
                            │ │ A 的上下文    │  │
                            │ │ B 的上下文    │  │
                            │ └──────────────┘  │
                            │ ┌──────────────┐  │
                            │ │ 工具权限隔离  │  │
                            │ │ A 的文件沙箱  │  │
                            │ │ B 的文件沙箱  │  │
                            │ └──────────────┘  │
                            └──────────────────┘
```

**关键原则**：
- **认证在最外层**：API Gateway 做身份验证，不让未认证请求进入 Agent
- **Session 级隔离**：每个用户的上下文、记忆、工具调用结果严格隔离
- **权限最小化**：Agent 能访问的工具和数据由用户权限决定
- **审计日志**：每次工具调用记录 who/what/when/result

### 挑战 5: 可观测性 — 你得知道 Agent 在想什么

```
传统服务的可观测性        Agent 服务的可观测性
──────────────────      ─────────────────
HTTP 状态码              + Agent 决策路径
响应时间                 + 每步推理耗时
错误率                   + Token 消耗量
吞吐量                   + 工具调用成功率
                         + 模型选择日志
                         + 上下文窗口使用率
                         + 幻觉检测指标
```

**实现**：
- OpenTelemetry 集成，Trace 贯穿整个 Agent 调用链
- 每个工具调用作为一个 Span
- Agent 的"思考过程"作为 Event 记录（不记原文，记摘要）
- 成本仪表盘：按用户、按任务类型、按模型统计 token 消耗

---

## 四、推荐的云端 Agent 服务架构

### 整体架构

```
                    ┌──────────────────────────────┐
                    │          客户端               │
                    │  Web / App / API Consumer    │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │        API Gateway           │
                    │  认证 · 限流 · 路由 · 计量     │
                    └──────────────┬───────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
    ┌─────────▼──────┐  ┌─────────▼──────┐  ┌─────────▼──────┐
    │  Routing Layer  │  │ Agent Runtime  │  │  Webhook /     │
    │  简单请求直接    │  │ 复杂任务走     │  │  Async Tasks   │
    │  LLM 调用       │  │ Agent 流程     │  │  长任务异步化   │
    └────────────────┘  └───────┬────────┘  └────────────────┘
                                │
                    ┌───────────┼───────────┐
                    │           │           │
              ┌─────▼───┐ ┌────▼────┐ ┌────▼────┐
              │ Worker A │ │Worker B │ │Worker C │
              │ (搜索)   │ │(分析)   │ │(生成)   │
              └─────┬───┘ └────┬────┘ └────┬────┘
                    │          │           │
    ┌───────────────┴──────────┴───────────┴───────────┐
    │                    基础设施层                       │
    │                                                    │
    │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
    │  │  Redis   │  │ 对象存储  │  │  向量数据库       │ │
    │  │ 会话状态  │  │ 文件/结果 │  │  长期记忆/RAG    │ │
    │  │ 语义缓存  │  │ S3/MinIO │  │  Pgvector/Milvus │ │
    │  └──────────┘  └──────────┘  └──────────────────┘ │
    │                                                    │
    │  ┌──────────────────────────────────────────────┐ │
    │  │  可观测性: OpenTelemetry + 成本仪表盘          │ │
    │  └──────────────────────────────────────────────┘ │
    └──────────────────────────────────────────────────┘
```

### 关键设计决策

| 决策点 | 推荐方案 | 理由 |
|--------|----------|------|
| 计算模型 | 容器 (ECS/K8s) 为主，Serverless 为辅 | Agent 需要长连接和状态，Lambda 15 分钟超时太短 |
| 状态存储 | Redis (热) + PostgreSQL (温) + S3 (冷) | 三温度分层，平衡性能和成本 |
| 异步处理 | 消息队列 (SQS/Redis Stream) | 长任务不阻塞 HTTP 请求，支持重试 |
| 模型调用 | 统一 Gateway + 模型路由 | 方便切换模型、A/B 测试、成本控制 |
| 部署策略 | 金丝雀 / Rainbow 部署 | 不中断运行中的 Agent 会话 |

---

## 五、从简单到复杂的演进路径

不要一上来就搞多 Agent。Anthropic 的建议：

```
阶段 1: 单 Agent + Prompt Chaining
├── 单一 LLM 调用 + 固定步骤串联
├── 状态存 Redis，结果存 S3
├── 适合：MVP、验证产品想法
└── 成本：最低

    ↓ 当单一 prompt 无法处理所有类型输入时

阶段 2: Routing + 模型分层
├── 分类输入，走不同处理路径
├── 简单任务用小模型，复杂任务用大模型
├── 适合：输入多样化，成本敏感
└── 成本：显著降低（简单请求不烧大模型 token）

    ↓ 当单步处理质量不够时

阶段 3: Evaluator-Optimizer / Parallelization
├── 生成后自动评估，不合格就重写
├── 或多路并行取最优
├── 适合：质量要求高的场景
└── 成本：2-3 倍

    ↓ 当任务复杂到无法预定义步骤时

阶段 4: Orchestrator-Workers
├── 动态分解子任务 + 多 Worker 并行
├── 需要完整的状态管理 + 检查点 + 可观测性
├── 适合：真正需要 Agent 自主决策的场景
└── 成本：5-15 倍，需要 FinOps 管控
```

---

## 六、实战 Checklist

上线前逐项检查：

**基础**
- [ ] 状态外部化（Redis/DB），实例无状态
- [ ] API 认证 + 用户级限流
- [ ] 每个请求设 token 上限和超时
- [ ] 错误返回友好信息，不暴露内部细节

**成本**
- [ ] 语义缓存（相似问题不重复调用 LLM）
- [ ] 模型路由（简单任务用小模型）
- [ ] 按用户/任务类型统计 token 消耗
- [ ] 设置成本告警阈值

**可靠性**
- [ ] 关键步骤有检查点，失败可恢复
- [ ] 工具调用有重试 + 熔断
- [ ] 长任务异步化 + 进度通知
- [ ] 降级策略（Agent 失败时的兜底方案）

**安全**
- [ ] 多租户数据隔离
- [ ] Agent 工具调用走权限控制
- [ ] 敏感操作需人工确认
- [ ] 完整审计日志

**可观测性**
- [ ] 全链路 Tracing（请求 → Agent 推理 → 工具调用 → 回复）
- [ ] Token 消耗仪表盘
- [ ] Agent 决策路径可回溯
- [ ] 异常模式告警

---

## 参考资料

- [Building Effective Agents — Anthropic](https://www.anthropic.com/research/building-effective-agents)
- [Anthropic Agents Cookbook — GitHub](https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents)
- [AI Agent Architecture: Build Systems That Work in 2026 — Redis](https://redis.io/blog/ai-agent-architecture/)
- [Effectively building AI agents on AWS Serverless — AWS](https://aws.amazon.com/blogs/compute/effectively-building-ai-agents-on-aws-serverless/)
- [How we built our multi-agent research system — Anthropic](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Multi-agent workflows often fail — GitHub Blog](https://github.blog/ai-and-ml/generative-ai/multi-agent-workflows-often-fail-heres-how-to-engineer-ones-that-dont/)
- [Building Scalable AI Agents on Google Cloud](https://cloud.google.com/blog/topics/partners/building-scalable-ai-agents-design-patterns-with-agent-engine-on-google-cloud)
- [AI agent orchestration platforms compared — Redis](https://redis.io/blog/ai-agent-orchestration-platforms/)
