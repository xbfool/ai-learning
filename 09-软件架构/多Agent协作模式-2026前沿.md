# 多 Agent 协作模式 — 2026 前沿探索

> 更新：2026-03-16
> CWD（Coordinator/Worker/Delegator）是 2025 年的经典模式。2026 年，以 Claude 为代表的实践者已经把这个模式推进到了新阶段。

---

## 一、从 CWD 到五大编排模式

CWD 是多 Agent 协作的起点，但实践中已演化出更细分的模式：

```
CWD (2025 经典)
  Coordinator ─── Worker ─── Delegator

2026 五大模式（Claude Code Agent Teams 实践总结）
  ├── Leader    — 分层指挥，Team Lead 分配任务给 Teammates
  ├── Swarm     — 群体并行，多 Agent 独立处理同类任务
  ├── Pipeline  — 流水线，Agent A 的输出是 Agent B 的输入
  ├── Council   — 议会制，多 Agent 从不同角度辩论后达成共识
  └── Watchdog  — 监督者，专门监控其他 Agent 的质量和行为
```

### 对照关系

| CWD 角色 | 2026 模式 | 进化点 |
|----------|-----------|--------|
| Coordinator | Leader | 增加了 Task List 依赖追踪、Mailbox 双向通信 |
| Worker | Swarm / Pipeline | 区分了"并行同构"和"串行异构"两种执行模式 |
| Delegator | Leader + Council | 委派不只是转发，还包括辩论和共识 |
| （无） | Watchdog | 新增：专门的质量监控角色 |

---

## 二、Claude 的实践：三个层次

### 层次 1: Claude Agent SDK — 单 Agent 增强

核心循环：**Gather Context → Take Action → Verify Work → Repeat**

不同于传统框架的预定义 workflow，Claude Agent SDK 的核心理念是：

> "给 Claude 一台电脑"——Agent 自己决定怎么做，而不是开发者预定义每一步。

关键设计：
- **文件系统即上下文工程** — 目录结构决定 Agent 能看到什么信息
- **代码生成作为可组合输出** — Agent 写脚本来处理复杂逻辑，比 JSON 管道更灵活
- **Context Compaction** — 长时间运行时自动压缩上下文，防止窗口溢出

### 层次 2: Claude Code Agent Teams — 多 Agent 协调

2026 年 Anthropic 发布的 Agent Teams（研究预览），架构：

```
┌──────────────────────────────────────┐
│            Team Lead                 │
│  - 分解任务                          │
│  - 分配给 Teammates                  │
│  - 汇总结果                          │
└───────┬──────────┬──────────┬────────┘
        │          │          │
   ┌────▼───┐ ┌───▼────┐ ┌───▼────┐
   │Teammate│ │Teammate│ │Teammate│
   │ Agent A│ │ Agent B│ │ Agent C│
   │(前端)  │ │(后端)  │ │(测试)  │
   └────────┘ └────────┘ └────────┘
        ↕          ↕          ↕
   独立上下文   独立上下文   独立上下文
```

**为什么要独立上下文？** 因为 LLM 的性能随上下文扩大而下降。每个 Teammate 只看到自己需要的信息，推理质量更高。

关键机制：
- **共享 Task List** — 有依赖追踪和自动解锁，A 完成后 B 自动开始
- **Mailbox 系统** — Teammate 之间可以直接通信，不必全部经过 Lead
- **文件所有权** — 每个 Teammate 负责不同文件，避免合并冲突
- **Delegate 模式** — 防止 Lead 自己动手干活，强制委派

实战数据：
- Anthropic 用 **16 个 Claude 实例**并行，写出 10 万行 Rust C 编译器（能编译 Linux 内核）
- 总计 ~2000 个 Claude Code 会话，20 亿输入 token，1.4 亿输出 token

### 层次 3: Anthropic Research 系统 — 生产级多 Agent

Anthropic 自己的 Research 功能，是目前公开最详细的生产级多 Agent 案例：

```
用户提问
   │
   ▼
┌──────────────┐
│ Lead Agent   │ ← 制定研究计划，存入外部记忆
│ (Opus 4.6)   │
└──┬───┬───┬───┘
   │   │   │
   ▼   ▼   ▼      ← 并行派出 3-10+ 个 Subagent
┌───┐┌───┐┌───┐
│ S1││ S2││ S3│   每个 Subagent：独立搜索 → 评估 → 返回发现
└─┬─┘└─┬─┘└─┬─┘
  │    │    │
  └────┼────┘
       ▼
┌──────────────┐
│ Lead Agent   │ ← 汇总，决定是否需要追加研究
│ 综合 + 判断   │
└──────┬───────┘
       ▼
┌──────────────┐
│ CitationAgent│ ← 专门做引用标注
└──────────────┘
       ▼
     最终报告
```

**关键数据**：
- 多 Agent 比单 Agent Opus 4 表现好 **90.2%**
- 性能方差的 95% 由三个因素解释：token 用量(80%)、工具调用频率、模型选择
- **模型质量的提升 > 单纯增加 token 预算**
- 成本：Agent 用量 ≈ 普通对话的 4 倍，多 Agent ≈ 15 倍

**八大工程原则**（Anthropic 总结）：

| # | 原则 | 核心内容 |
|---|------|----------|
| 1 | 心智模型 | 通过逐步模拟理解 Agent 的真实行为，不要想当然 |
| 2 | 委派框架 | 明确目标、输出格式、工具指引、任务边界，防止重复劳动 |
| 3 | 努力缩放 | 简单问题 1 个 Agent + 3-10 次调用；复杂研究 10+ Subagent |
| 4 | 工具人机工程学 | Agent-Tool 接口和人机界面一样重要，描述清晰决定能否正确使用 |
| 5 | 自我改进 | Claude 4 能自我诊断失败并改进工具描述，任务时间降低 40% |
| 6 | 搜索策略 | 先广后窄，模仿人类专家的研究模式 |
| 7 | Extended Thinking | 把思考过程当作可控的草稿纸，用于规划和评估 |
| 8 | 并行化 | 3-5 个 Subagent 同时跑 + 并行工具调用，研究时间降低 90% |

---

## 三、2026 的新认知：Agent = 分布式系统

GitHub 工程团队在 2026 年 2 月发布了一篇重要文章，核心观点：

> **"把 Agent 当分布式系统来设计，不要当聊天流。"**

### 多 Agent 系统的三大失败模式

| 失败模式 | 表现 | 根因 |
|----------|------|------|
| 共享状态冲突 | Agent A 关了 Agent B 刚开的 issue | 没有状态锁 / 事务机制 |
| 顺序依赖 | 提交了代码但下游检查不知道 | 缺少显式依赖声明 |
| 数据格式漂移 | JSON 字段名变了，下游解析失败 | 没有 Schema 约束 |

### 三大工程模式（对应分布式系统经典方案）

**1. Typed Schema — 数据契约**
```
// 不是让 Agent 自由发挥 JSON，而是强制类型
type UserProfile = {
  id: number;
  email: string;
  plan: "free" | "pro" | "enterprise";
};
```
→ 对应分布式系统中的 **接口定义 (IDL)** / **Protobuf**

**2. Action Schema — 行动约束**
- 不说"分析并帮助团队"，而是定义可选动作集
- Agent 只能从预定义的动作中选择
- → 对应分布式系统中的 **有限状态机 (FSM)**

**3. MCP 作为执行层 — 契约强制执行**
- MCP 验证每个工具调用的输入/输出是否符合 Schema
- 不合格的调用直接拒绝，不让坏数据传播
- → 对应分布式系统中的 **API Gateway 校验**

### Anthropic 生产系统的工程策略

| 策略 | 说明 | 对应传统分布式概念 |
|------|------|-------------------|
| 外部记忆 | Agent 把计划存到外部，新 Subagent 从记忆恢复 | 持久化 + 断点续跑 |
| Rainbow 部署 | 流量逐步切换到新版本，不中断运行中的 Agent | 金丝雀部署 |
| 全链路追踪 | 记录决策模式，不记录对话内容 | 分布式 Tracing (Jaeger/Zipkin) |
| 重试 + 检查点 | 从失败点恢复，不从头开始 | Saga 补偿 / WAL |

---

## 四、对我学习架构的启示

```
传统软件架构知识                      Agent 架构中的映射
──────────────                      ────────────────
微服务拆分            →              Agent 职责划分
API Gateway           →              Coordinator / Team Lead
服务发现              →              Agent 注册 + 能力描述
消息队列              →              Mailbox / Task List
接口契约 (IDL)        →              Typed Schema + MCP
熔断/限流             →              Token 预算 + 超时
分布式事务 (Saga)     →              Agent 重试 + 检查点
可观测性              →              Agent Tracing
金丝雀部署            →              Rainbow Deployment
```

**核心洞察**：学好 DDIA 和分布式系统基础，就能直接迁移到 Agent 架构设计。两者是同一套思维模型的不同实例化。

---

## 参考资料

- [How we built our multi-agent research system — Anthropic Engineering](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Building agents with the Claude Agent SDK — Anthropic](https://claude.com/blog/building-agents-with-the-claude-agent-sdk)
- [Claude Code Agent Teams — Addy Osmani](https://addyosmani.com/blog/claude-code-agent-teams/)
- [Claude Code's Hidden Multi-Agent System — paddo.dev](https://paddo.dev/blog/claude-code-hidden-swarm/)
- [Multi-agent workflows often fail — GitHub Blog](https://github.blog/ai-and-ml/generative-ai/multi-agent-workflows-often-fail-heres-how-to-engineer-ones-that-dont/)
- [Claude Code Agent Teams Docs](https://code.claude.com/docs/en/agent-teams)
- [Agent Design Patterns — Lance Martin](https://rlancemartin.github.io/2026/01/09/agent_design/)
