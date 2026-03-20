# hotel_management_workflows.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
#!/usr/bin/env python3
"""Sequential Workflow Demo: Hotel Search → Hotel Booking"""

import asyncio

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.models.openai import OpenAIChat
from agno.tools.mcp_toolbox import MCPToolbox
from agno.workflow.condition import Step
from agno.workflow.workflow import Workflow

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


# Configuration
url = "http://127.0.0.1:5001"

# Database for workflow
db = SqliteDb(db_file="tmp/workflow_demo.db")

# Create agents with different toolsets
search_agent = Agent(
    name="Hotel Search Agent",
    model=OpenAIChat(id="gpt-5.2"),
    instructions=[
        "You are a hotel search expert. Find hotels based on user requirements.",
        "Always provide hotel IDs, names, locations, and availability.",
        "Be specific about which hotels are available for booking.",
    ],
)

booking_agent = Agent(
    name="Hotel Booking Agent",
    model=OpenAIChat(id="gpt-5.2"),
    instructions=[
        "You are a booking specialist. Book hotels using the hotel IDs provided.",
        "Always confirm successful bookings with hotel name and ID.",
        "If booking fails, explain the reason clearly.",
    ],
)

# Define workflow steps
search_step = Step(
    name="Search Hotels",
    agent=search_agent,
)

booking_step = Step(
    name="Book Hotel",
    agent=booking_agent,
)

# Create the workflow
workflow = Workflow(
    name="hotel-workflow",
    description="Search and book hotels sequentially",
    db=db,
    steps=[search_step, booking_step],
)


async def run_workflow_demo():
    """Run the hotel workflow with MCP toolboxes"""

    # Create separate toolboxes for each agent's role
    search_tools = MCPToolbox(url=url, toolsets=["hotel-management"])
    booking_tools = MCPToolbox(url=url, toolsets=["booking-system"])

    async with search_tools, booking_tools:
        # Assign tools to agents
        search_agent.tools = [search_tools]
        booking_agent.tools = [booking_tools]

        # Input for the workflow
        user_request = "Find luxury hotels in Zurich and book the first available one"

        print("Hotel Search and Booking Workflow")
        print(f"Request: {user_request}")
        print("=" * 50)

        # Execute workflow
        result = await workflow.arun(user_request)

        print("\nWorkflow Result:")
        print(f"Content: {result.content}")
        print(f"Steps executed: {len(result.step_results)}")

        return result


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    asyncio.run(run_workflow_demo())
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/mcp/mcp_toolbox_demo/hotel_management_workflows.py`

## 概述

Sequential Workflow Demo: Hotel Search → Hotel Booking

本示例归类：**Workflow**；模型相关类型：`OpenAIChat`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | 'Hotel Search Agent' | `Agent(...)` |
| `model` | OpenAIChat(id='gpt-5.2'…) | `Agent(...)` |
| （Model 类） | `OpenAIChat` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ hotel_management_workflows.py │  ──▶  │ Agent → get_run_messages → Model │
└──────────────────────┘         └────────────────────────────────┘
                                          │
                                          ▼
                                  ┌───────────────┐
                                  │ 对应 Model 子类 │
                                  └───────────────┘
```

## 核心组件解析

### 运行机制与因果链

1. **入口**：从模块 `__main__` 或暴露的 `agent` / `team` 调用进入；同步用 `print_response` / `run`，异步用 `aprint_response` / `arun`（若源码中有）。
2. **消息**：默认路径下 system 内容由 `get_system_message()`（`libs/agno/agno/agent/_messages.py` 约 **L106** 起）按分段逻辑拼装；若显式传入 `system_message` 则早退使用该字符串。
3. **模型**：具体 HTTP/SDK 形态以 `libs/agno/agno/models/` 下对应类的 `invoke` / `ainvoke` 为准（勿默认写成单一 `chat.completions`）。
4. **副作用**：若配置 `db`、`knowledge`、`memory`，运行会读写存储；仅以本文件为准对照。

### 与框架的衔接

- **System**：`get_system_message()` 锚点 `agno/agent/_messages.py` **L106+**。
- **运行**：`Agent.print_response` 等入口 `agno/agent/agent.py`（以当前仓库检索为准）。

## System Prompt 组装

| 序号 | 组成部分 | 本文件 | 是否生效 |
|------|---------|--------|---------|
| 1 | `instructions` / `description` 等 | 见核心配置表与源码 | 有赋值则生效 |
| 2 | 默认分段（markdown、时间等） | 取决于 `Agent` 默认与显式参数 | 视参数 |

### 拼装顺序与源码锚点

1. `system_message` 直给 → 使用该内容（见 `_messages.py` 文档字符串分支说明）。
2. 否则默认拼装：`description`、`role`、`instructions`、markdown 附加段等按 `# 3.x` 注释顺序合并。

### 还原后的完整 System 文本

```text
（主 `Agent(...)` 未传入可静态解析的 `description`/`instructions`/`system_message` 字符串；此时 system 由 `get_system_message()` 默认段与 `markdown` 等开关决定，请在 `agno/agent/_messages.py` 对照分段注释，或在运行中打印 `get_system_message` 返回值。）
```

### 段落释义（模型视角）

- 指令与安全边界由 `instructions` / `system_message` 约束；若带 `tools` / `knowledge`，文档中需体现「何时检索/调用」由框架注入的提示段支持。

## 完整 API 请求

```python
# 请以本文件实际 Model 为准打开 libs/agno/agno/models/<厂商>/ 下对应类的 invoke：
# 可能是 chat.completions.create、responses.create、Gemini generate_content 等。
```

> 与上一节 system 文本在同一 run 中组合；`developer`/`system` 角色由适配器转换。

```mermaid
flowchart TD
    Entry["用户入口<br/>`if __name__` / main"] --> Run["【关键】Agent.run / print_response"]
    Run --> Sys["【关键】get_system_message<br/>agno/agent/_messages.py L106+"]
    Sys --> Inv["【关键】Model.invoke / 提供商 API"]
    Inv --> Out["RunOutput / 流式 chunk"]
```

**【关键】节点说明：**

- **print_response / run**：用户可见的同步入口。
- **get_system_message**：系统提示拼装核心。
- **Model.invoke**：对模型提供商的实际请求。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/_messages.py` | `get_system_message()` L106+ |
| `agno/agent/agent.py` | `Agent` 运行与 CLI 输出 |
| `agno/models/` | 各厂商 `Model.invoke` |
