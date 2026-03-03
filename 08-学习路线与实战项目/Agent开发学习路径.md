# Agent 开发学习路径

> 14 个由浅入深的 Agent 实战项目，从第一个 API 调用到生产级 Agent 系统。约 3-5 个月完成。

---

## 阶段一：入门（2-3 周）

### 项目 1：我的第一个 AI Chatbot ★☆☆☆☆

> 用 API 构建命令行聊天机器人，理解 LLM 调用的基本模式。

| 属性 | 详情 |
|------|------|
| 预计时间 | 2-3 天 |
| 技术栈 | Python, OpenAI SDK 或 Anthropic SDK |
| 硬件需求 | 任意电脑 + API Key（费用约 $1-5） |
| 学到什么 | API 调用模式、消息格式（system/user/assistant）、流式输出、上下文管理 |

**核心代码**：
```python
from openai import OpenAI
client = OpenAI()

messages = [{"role": "system", "content": "你是一个有帮助的助手"}]

while True:
    user_input = input("你: ")
    messages.append({"role": "user", "content": user_input})

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=messages,
        stream=True  # 流式输出
    )

    assistant_msg = ""
    for chunk in response:
        if chunk.choices[0].delta.content:
            print(chunk.choices[0].delta.content, end="")
            assistant_msg += chunk.choices[0].delta.content
    print()
    messages.append({"role": "assistant", "content": assistant_msg})
```

**教程**：
- [OpenAI API 快速入门](https://platform.openai.com/docs/quickstart)
- [Anthropic Claude API 快速入门](https://docs.anthropic.com/en/docs/quickstart)

---

### 项目 2：给 Chatbot 加工具调用 ★★☆☆☆

> 让 Chatbot 学会调用外部工具（天气、计算器、搜索），理解 Function Calling。

| 属性 | 详情 |
|------|------|
| 预计时间 | 3-4 天 |
| 技术栈 | Python, OpenAI SDK (tools 参数) |
| 硬件需求 | 任意电脑 + API Key |
| 学到什么 | Function Calling 完整流程、JSON Schema、工具编排、Agent 雏形 |

**核心概念**：
```
传统 Chatbot: 用户 → 模型 → 回答（纯文本）
工具增强:     用户 → 模型 → "我需要调用工具" → 执行工具 → 模型整合结果 → 回答

这就是 Agent 的核心：LLM + 工具 = Agent
```

**教程**：
- [OpenAI Function Calling 指南](https://platform.openai.com/docs/guides/function-calling)
- [DataCamp: Function Calling 教程](https://www.datacamp.com/tutorial/open-ai-function-calling-tutorial)

---

### 项目 3：我的第一个 MCP Server ★★☆☆☆

> 开发一个 MCP Server，暴露自定义工具给 Claude Desktop 使用。

| 属性 | 详情 |
|------|------|
| 预计时间 | 3-5 天 |
| 技术栈 | TypeScript 或 Python, MCP SDK, Claude Desktop |
| 硬件需求 | 任意电脑 |
| 学到什么 | MCP 协议、Tool/Resource/Prompt 三大原语、Server 开发 |

**项目创意**：
- 做一个 "笔记管理" MCP Server（创建/搜索/删除笔记）
- 做一个 "天气查询" MCP Server（调用天气 API）
- 做一个 "数据库查询" MCP Server（连接 SQLite）

**教程**：
- [MCP 官方: Build a Server](https://modelcontextprotocol.io/docs/develop/build-server)
- [Microsoft MCP for Beginners](https://github.com/microsoft/mcp-for-beginners) — 多语言教程
- [freeCodeCamp: MCP Server with TypeScript](https://www.freecodecamp.org/news/how-to-build-a-custom-mcp-server-with-typescript-a-handbook-for-developers/)

---

### 项目 4：用 Smolagents 写第一个 Agent ★★☆☆☆

> 用 HuggingFace 的 Smolagents 框架，构建一个能自主推理和调用工具的 Agent。

| 属性 | 详情 |
|------|------|
| 预计时间 | 3-4 天 |
| 技术栈 | Python, smolagents |
| 硬件需求 | 任意电脑 + API Key |
| 学到什么 | Agent 循环（思考→行动→观察→反思）、CodeAgent、ReAct 范式 |

**为什么用 Smolagents**：
- HuggingFace 出品，极简设计，代码少
- 理解 Agent 核心原理的最佳教学框架
- CodeAgent 模式：模型写 Python 代码执行（而非 JSON 调用）

**教程**：
- [HuggingFace AI Agents Course](https://huggingface.co/learn/agents-course/en/unit0/introduction) — **强烈推荐的免费课程**
- [Smolagents 官方文档](https://huggingface.co/docs/smolagents/en/index)

---

## 阶段二：进阶（5-7 周）

### 项目 5：LangGraph 多步骤工作流 Agent ★★★☆☆

> 用 LangGraph 的图结构构建具有条件分支、循环和状态管理的 Agent。

| 属性 | 详情 |
|------|------|
| 预计时间 | 5-7 天 |
| 技术栈 | Python, LangGraph, LangChain |
| 硬件需求 | 任意电脑 + API Key |
| 学到什么 | 图结构 Agent、状态管理、条件路由、Human-in-the-Loop |

**核心概念**：
```
LangGraph 把 Agent 行为建模为有限状态机：

  [规划] ──→ [执行] ──→ [检查]
     ↑                      │
     └──── 需要重新规划 ←────┘
                            │
                         [完成] → 输出
```

**教程**：
- [LangChain Academy](https://academy.langchain.com/) — LangGraph 官方课程
- [freeCodeCamp: LangGraph 实战指南](https://www.freecodecamp.org/news/how-to-develop-ai-agents-using-langgraph-a-practical-guide/)

---

### 项目 6：CrewAI 多 Agent 协作 ★★★☆☆

> 创建一个 "AI 团队"，多个角色 Agent 协作完成任务（如内容创作流水线）。

| 属性 | 详情 |
|------|------|
| 预计时间 | 5-7 天 |
| 技术栈 | Python, CrewAI |
| 硬件需求 | 任意电脑 + API Key |
| 学到什么 | 多 Agent 架构、角色定义、任务编排、Agent 间通信 |

**项目创意**：内容创作团队
```
Researcher Agent → 搜索资料
Writer Agent     → 撰写初稿
Editor Agent     → 审查修改
Publisher Agent  → 格式化发布
```

**教程**：
- [CrewAI 官方文档](https://docs.crewai.com/)
- [DigitalOcean: CrewAI 速成教程](https://www.digitalocean.com/community/tutorials/crewai-crash-course-role-based-agent-orchestration)

---

### 项目 7：RAG Agent ★★★☆☆

> 构建一个 Agentic RAG 系统，能自主检索、评估、重新查询直到找到满意答案。

| 属性 | 详情 |
|------|------|
| 预计时间 | 7-10 天 |
| 技术栈 | Python, LangChain 或 LlamaIndex, ChromaDB, Embedding 模型 |
| 硬件需求 | 任意电脑 + API Key |
| 学到什么 | 向量数据库、文档切分、Embedding、检索策略、Agentic RAG |

**项目创意**：把本学习文档做成知识库
```
1. 将 01-07 章的 .md 文件加载到向量数据库
2. 构建一个 Agent 可以回答"什么是 MLA？"这类问题
3. Agent 自主判断：答案够不够好？要不要换个搜索策略？
```

**教程**：
- [DataCamp: Agentic RAG 教程](https://www.datacamp.com/tutorial/agentic-rag-tutorial)
- [LangChain RAG 文档](https://docs.langchain.com/oss/python/langchain/rag)

---

### 项目 8：Deep Research 搜索 Agent ★★★★☆

> 实现 Open Deep Research 风格的搜索 Agent，自主多轮搜索、阅读、生成研究报告。

| 属性 | 详情 |
|------|------|
| 预计时间 | 7-10 天 |
| 技术栈 | Python, LangGraph, 搜索 API (Tavily/Serper) |
| 硬件需求 | 任意电脑 + API Key |
| 学到什么 | 多轮搜索策略、信息综合、引用追踪、报告生成 |

**核心架构**：
```
问题 → 查询分解 → 搜索 → 阅读 → 信息够了吗？
                                         │
                     ┌─── 不够 ───────────┘
                     │
                     ▼
              重新构造查询 → 搜索 → 阅读 → ...
                                         │
                     ┌─── 够了 ───────────┘
                     ▼
              综合信息 → 生成报告
```

**教程**：
- [LangChain Open Deep Research (GitHub)](https://github.com/langchain-ai/open_deep_research) — 最佳参考实现
- [HuggingFace: Open-source DeepResearch](https://huggingface.co/blog/open-deep-research)

---

### 项目 9：代码审查 Agent ★★★★☆

> 开发一个能阅读 Git Diff、分析代码质量、提出改进建议的 Agent。

| 属性 | 详情 |
|------|------|
| 预计时间 | 7-10 天 |
| 技术栈 | Python/TypeScript, GitHub API, Claude/GPT API |
| 硬件需求 | 任意电脑 + API Key |
| 学到什么 | 代码分析、Git 集成、结构化输出、大上下文处理 |

**教程**：
- [PR-Agent by CodiumAI (GitHub)](https://github.com/Codium-ai/pr-agent) — 参考实现

---

## 阶段三：挑战（6-10 周，可选）

### 项目 10：OpenAI Agents SDK 多 Agent 服务 ★★★★☆

> 用 OpenAI Agents SDK 构建多 Agent 系统，实现 Handoff 和 Guardrails。

| 属性 | 详情 |
|------|------|
| 预计时间 | 7-10 天 |
| 技术栈 | Python, openai-agents SDK, FastAPI |
| 学到什么 | Agent Handoff（任务移交）、Guardrails（安全护栏）、Tracing |

**教程**：
- [OpenAI Agents SDK 官方文档](https://openai.github.io/openai-agents-python/)
- [GitHub: openai-agents-python](https://github.com/openai/openai-agents-python)

---

### 项目 11：Claude Agent SDK 完整 Agent 服务 ★★★★☆

> 用 Anthropic Claude Agent SDK 开发连接 MCP 工具的完整 Agent 服务。

| 属性 | 详情 |
|------|------|
| 预计时间 | 7-10 天 |
| 技术栈 | Python/TypeScript, Claude Agent SDK, MCP Servers, FastAPI |
| 学到什么 | Claude Agent SDK 架构、MCP 集成、Agent 服务化 |

**教程**：
- [Claude Agent SDK 官方文档](https://platform.claude.com/docs/en/agent-sdk/overview)
- [Nader Dabit: Building Agents with Claude Agent SDK](https://nader.substack.com/p/the-complete-guide-to-building-agents)

---

### 项目 12：Computer Use Agent ★★★★★

> 让 AI 看屏幕、点鼠标、打键盘，构建桌面自动化 Agent。

| 属性 | 详情 |
|------|------|
| 预计时间 | 10-14 天 |
| 技术栈 | Python, Claude API (computer-use tool), Docker |
| 硬件需求 | 16GB+ RAM |
| 学到什么 | Computer Use API、截图-决策循环、安全沙箱 |

**教程**：
- [Claude Computer Use 官方文档](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool)
- [Anthropic Quickstart: Computer Use Demo](https://github.com/anthropics/anthropic-quickstarts/tree/main/computer-use-demo)

---

### 项目 13：用 RL 训练一个 Agent ★★★★★

> 用 GRPO 算法对小模型做强化学习训练，让它学会自主使用工具。

| 属性 | 详情 |
|------|------|
| 预计时间 | 14-21 天 |
| 技术栈 | Python, Unsloth 或 TRL, GRPO |
| 硬件需求 | **RTX 3060 12GB 最低**，建议 RTX 4090 |
| 学到什么 | GRPO 算法、奖励函数设计、Agent RL 训练 |

**教程**：
- [Unsloth RL 训练指南 (GRPO)](https://unsloth.ai/docs/get-started/reinforcement-learning-rl-guide)
- [OpenPipe ART: Agent Reinforcement Trainer](https://github.com/OpenPipe/ART)
- [Cameron Wolfe: GRPO 深度解析](https://cameronrwolfe.substack.com/p/grpo)

---

### 项目 14：生产级 Agent 系统 ★★★★★

> 将之前的 Agent 项目升级为生产级服务，加上监控、评估、安全全套基础设施。

| 属性 | 详情 |
|------|------|
| 预计时间 | 14-21 天 |
| 技术栈 | FastAPI, Docker, LangSmith/Langfuse, PostgreSQL |
| 学到什么 | Agent 服务化、可观测性、评估体系、安全防护、成本控制 |

**关键组件**：
```
Agent 生产化清单：
☐ HTTP API + WebSocket 流式输出
☐ Tracing（追踪每个 Agent 步骤）— LangSmith 或 Langfuse
☐ 输入验证 + 输出过滤（安全）
☐ Token 预算 + 缓存（成本控制）
☐ Docker 化 + 健康检查（部署）
☐ A/B 测试框架（评估）
```

**教程**：
- [LangSmith 文档](https://docs.smith.langchain.com/)
- [Langfuse (开源替代)](https://langfuse.com/)

---

## 时间线总览

```
阶段一入门（2-3 周）：
  项目1 [API] → 项目2 [工具调用] → 项目3 [MCP] → 项目4 [Smolagents]

阶段二进阶（5-7 周）：
  项目5 [LangGraph] → 项目6 [CrewAI] → 项目7 [RAG] → 项目8 [搜索Agent] → 项目9 [代码Agent]

阶段三挑战（6-10 周，可选）：
  项目10 [OpenAI SDK] → 项目11 [Claude SDK] → 项目12 [Computer Use] → 项目13 [RL] → 项目14 [生产部署]
```

**建议**：
- 入门阶段不要跳过，打好 API 调用和工具调用的基础
- 每个项目推到 GitHub，写 README 记录学到什么
- 项目 13（RL 训练）可根据 GPU 情况选做
- 项目 14 可以选择之前任意一个项目升级
