<div align="center" id="top">
  <a href="https://agno.com">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://agno-public.s3.us-east-1.amazonaws.com/assets/logo-dark.svg">
      <source media="(prefers-color-scheme: light)" srcset="https://agno-public.s3.us-east-1.amazonaws.com/assets/logo-light.svg">
      <img src="https://agno-public.s3.us-east-1.amazonaws.com/assets/logo-light.svg" alt="Agno">
    </picture>
  </a>
</div>

<p align="center">
  大规模构建、运行和管理 agentic 软件。
</p>

<div align="center">
  <a href="https://docs.agno.com">文档</a>
  <span>&nbsp;•&nbsp;</span>
  <a href="https://github.com/agno-agi/agno/tree/main/cookbook">Cookbook</a>
  <span>&nbsp;•&nbsp;</span>
  <a href="https://docs.agno.com/first-agent">快速开始</a>
  <span>&nbsp;•&nbsp;</span>
  <a href="https://www.agno.com/discord">Discord</a>
</div>

## 什么是 Agno

Agno 是 agentic 软件的运行时。构建 Agent、团队和工作流，将它们作为可扩展服务运行，并在生产环境中监控和管理。

| 层级 | 功能 |
|-------|--------------|
| **框架（Framework）** | 构建带有记忆（memory）、知识库（knowledge）、护栏（guardrails）及 100+ 集成的 Agent、团队和工作流。 |
| **运行时（Runtime）** | 使用无状态、会话作用域的 FastAPI 后端在生产中提供服务。 |
| **控制平面（Control Plane）** | 使用 [AgentOS UI](https://os.agno.com) 测试、监控和管理系统。 |

## 快速开始

用约 20 行代码构建一个有状态、使用工具的 Agent，并将其作为生产 API 提供服务。

```python
from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.models.anthropic import Claude
from agno.os import AgentOS
from agno.tools.mcp import MCPTools

agno_assist = Agent(
    name="Agno Assist",
    model=Claude(id="claude-sonnet-4-6"),
    db=SqliteDb(db_file="agno.db"),
    tools=[MCPTools(url="https://docs.agno.com/mcp")],
    add_history_to_context=True,
    num_history_runs=3,
    markdown=True,
)

agent_os = AgentOS(agents=[agno_assist], tracing=True)
app = agent_os.get_app()
```

运行：

```bash
export ANTHROPIC_API_KEY="***"

uvx --python 3.12 \
  --with "agno[os]" \
  --with anthropic \
  --with mcp \
  fastapi dev agno_assist.py
```

约 20 行代码，你将获得：
- 具有流式响应的有状态 Agent
- 按用户、按会话的隔离
- http://localhost:8000 上的生产 API
- 原生链路追踪（tracing）

连接到 [AgentOS UI](https://os.agno.com) 来监控、管理和测试你的 Agent。

1. 打开 [os.agno.com](https://os.agno.com) 并登录。
2. 在顶部导航栏点击 **"Add new OS"**。
3. 选择 **"Local"** 连接到本地 AgentOS。
4. 输入端点 URL（默认：`http://localhost:8000`）。
5. 命名为 "Local AgentOS"。
6. 点击 **"Connect"**。

https://github.com/user-attachments/assets/75258047-2471-4920-8874-30d68c492683

打开 Chat，选择你的 Agent，然后提问：

> What is Agno?

Agent 从 Agno MCP 服务器检索上下文，并给出有依据的回答。

https://github.com/user-attachments/assets/24c28d28-1d17-492c-815d-810e992ea8d2

你可以使用完全相同的架构在生产中运行多 Agent 系统。

## 为什么选择 Agno？

Agentic 软件带来了三个根本性转变。

### 新的交互模型

传统软件接收请求并返回响应。Agent 实时流式输出推理过程、工具调用和结果。它们可以在执行中途暂停，等待批准，然后稍后恢复。

Agno 将流式传输和长时间运行的执行作为一等公民对待。

### 新的治理模型

传统系统执行事先编写好的预定义决策逻辑。Agent 动态选择行动。有些行动风险较低，有些需要用户批准，有些需要管理员权限。

Agno 让你将"谁决定什么"定义为 Agent 定义的一部分，包括：

- 审批工作流
- 人机协作（Human-in-the-loop）
- 审计日志
- 运行时强制执行

### 新的信任模型

传统系统被设计为可预测的，每条执行路径都事先定义好。Agent 将概率推理引入执行路径。

Agno 将信任内置到引擎本身：

- 护栏（Guardrails）作为执行的一部分运行
- 评估（Evaluations）集成到 Agent 循环中
- 链路追踪和审计日志作为一等公民

## 为生产而生

Agno 在你的基础设施中运行，而非我们的。

- 无状态、水平可扩展的运行时。
- 50+ API 和后台执行。
- 按用户和按会话的隔离。
- 运行时审批强制执行。
- 原生链路追踪和完整可审计性。
- Session、记忆（memory）、知识库（knowledge）和链路追踪存储在你的数据库中。

系统归你所有。数据归你所有。规则由你定义。

## 你能构建什么

Agno 驱动由上述相同原语构建的真实 agentic 系统。

- [**Pal →**](https://github.com/agno-agi/pal) 一个学习你偏好的个人 Agent。
- [**Dash →**](https://github.com/agno-agi/dash) 一个基于六层上下文的自学习数据 Agent。
- [**Scout →**](https://github.com/agno-agi/scout) 一个管理企业上下文知识的自学习上下文 Agent。
- [**Gcode →**](https://github.com/agno-agi/gcode) 一个随时间不断改进的后 IDE 编码 Agent。
- [**Investment Team →**](https://github.com/agno-agi/investment-team) 一个进行辩论和资本配置的多 Agent 投资委员会。

单个 Agent。协调团队。结构化工作流。全部构建在同一套架构之上。

## 开始使用

1. [阅读文档](https://docs.agno.com)
2. [构建你的第一个 Agent](https://docs.agno.com/first-agent)
3. 探索 [cookbook](https://github.com/agno-agi/agno/tree/main/cookbook)

## IDE 集成

将 Agno 文档作为数据源添加到你的编码工具中：

**Cursor：** Settings → Indexing & Docs → 添加 `https://docs.agno.com/llms-full.txt`

同样适用于 VSCode、Windsurf 及类似工具。

## 贡献

请参阅[贡献指南](https://github.com/agno-agi/agno/blob/main/CONTRIBUTING.md)。

## 遥测

Agno 记录使用了哪些模型提供商以优先安排更新。可通过设置 `AGNO_TELEMETRY=false` 禁用。

<p align="right"><a href="#top">↑ 返回顶部</a></p>
