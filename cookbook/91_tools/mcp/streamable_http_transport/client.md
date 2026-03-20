# client.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Show how to connect to MCP servers that use either SSE or Streamable HTTP transport using our MCPTools and MultiMCPTools classes.

Check the README.md file for instructions on how to run these examples.
"""

import asyncio

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.mcp import MCPTools, MultiMCPTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


# This is the URL of the MCP server we want to use.
server_url = "http://localhost:8000/mcp"


async def run_agent(message: str) -> None:
    mcp_tools = MCPTools(
        transport="streamable-http",
        url=server_url,
        refresh_connection=True,  # (Optional) Refresh the MCP connection and tools on each run
    )
    await mcp_tools.connect()
    agent = Agent(
        model=OpenAIChat(id="gpt-4o"),
        tools=[mcp_tools],
        markdown=True,
    )
    await agent.aprint_response(input=message, stream=True, markdown=True)
    await mcp_tools.close()


# Using MultiMCPTools, we can connect to multiple MCP servers at once, even if they use different transports.
# In this example we connect to both our example server (Streamable HTTP transport), and a different server (stdio transport).
async def run_agent_with_multimcp(message: str) -> None:
    mcp_tools = MultiMCPTools(
        commands=["npx -y @openbnb/mcp-server-airbnb --ignore-robots-txt"],
        urls=[server_url],
        urls_transports=["streamable-http"],
        refresh_connection=True,  # (Optional) Refresh the MCP connection and tools on each run
    )
    agent = Agent(
        model=OpenAIChat(id="gpt-4o"),
        tools=[mcp_tools],
        markdown=True,
    )
    await agent.aprint_response(input=message, stream=True, markdown=True)


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    asyncio.run(run_agent("Do I have any birthdays this week?"))
    asyncio.run(run_agent("What else is on my calendar this week?"))
    asyncio.run(
        run_agent_with_multimcp(
            "Can you check when is my mom's birthday, and if there are any AirBnb listings in SF for two people for that day?",
        )
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/mcp/streamable_http_transport/client.py`

## 概述

Show how to connect to MCP servers that use either SSE or Streamable HTTP transport using our MCPTools and MultiMCPTools classes.

本示例归类：**单 Agent**；模型相关类型：`OpenAIChat`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | OpenAIChat(id='gpt-4o'…) | `Agent(...)` |
| `markdown` | True | `Agent(...)` |
| （Model 类） | `OpenAIChat` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ client.py            │  ──▶  │ Agent → get_run_messages → Model │
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
