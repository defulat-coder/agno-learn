# client.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Agent with MCP tools using Dynamic Headers"""

import asyncio
from typing import TYPE_CHECKING, Optional

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.run import RunContext
from agno.tools.mcp import MCPTools

if TYPE_CHECKING:
    from agno.agent import Agent
    from agno.team import Team
# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


async def main():
    """Example showing dynamic headers with different users."""

    # Step 1: Define your header provider
    # This function can receive RunContext, Agent and/or Team based on its signature
    def header_provider(
        run_context: RunContext,
        agent: Optional["Agent"] = None,
        team: Optional["Team"] = None,
    ) -> dict:
        """
        Generate dynamic headers from RunContext and Agent.

        The header_provider can accept any combination of these parameters:
        - run_context: The RunContext for the current agent or team run
        - agent: The contextual Agent instance
        - team: The contextual Team instance

        The RunContext contains:
        - run_id: Unique ID for this agent run
        - user_id: User ID passed to agent.arun()
        - session_id: Session ID passed to agent.arun()
        - metadata: Dict of custom metadata passed to agent.arun()
        """
        headers = {
            "X-User-ID": run_context.user_id or "unknown",
            "X-Session-ID": run_context.session_id or "unknown",
            "X-Run-ID": run_context.run_id,
            "X-Tenant-ID": run_context.metadata.get("tenant_id", "no-tenant")
            if run_context.metadata
            else "no-tenant",
            # You can also access agent and team properties if needed
            "X-Agent-Name": agent.name
            if agent
            else team.name
            if team
            else "unnamed-agno-entity",
        }
        return headers

    # Step 2: Create MCPTools with header_provider
    # This enables dynamic headers for all MCP tool calls
    mcp_tools = MCPTools(
        url="http://localhost:8000/mcp",  # Your MCP server URL
        transport="streamable-http",  # Use streamable-http or sse for headers
        header_provider=header_provider,  # This enables dynamic headers!
    )

    # Step 3: Connect to MCP server
    await mcp_tools.connect()
    print("Connected to MCP server")
    print(f"   Available tools: {list(mcp_tools.functions.keys())}\n")

    try:
        # Step 4: Create agent with MCP tools
        agent_1 = Agent(
            name="agent-1",
            model=OpenAIChat(id="gpt-5.2"),
            tools=[mcp_tools],
            markdown=False,
        )

        # Step 5: Run agent with different users
        # The agent automatically creates RunContext and injects it into tools!

        # Example 1: User "neel"
        print("=" * 60)
        print("Example 1: Running as user 'neel'")
        print("=" * 60)

        response1 = await agent_1.arun(
            "Please use the greet tool to greet me. My name is neel.",
            user_id="neel",  # ← Goes into RunContext.user_id
            session_id="session-1",  # ← Goes into RunContext.session_id
            metadata={  # ← Goes into RunContext.metadata
                "tenant_id": "tenant-1",
            },
        )
        print(f"Response: {response1.content}\n")

        # Example 2: User "dirk"
        print("=" * 60)
        print("Example 2: Running as user 'dirk'")
        print("=" * 60)

        agent_2 = Agent(
            name="agent-2",
            model=OpenAIChat(id="gpt-5.2"),
            tools=[mcp_tools],
            markdown=False,
        )
        response2 = await agent_2.arun(
            "Please use the greet tool to greet me. My name is dirk.",
            user_id="dirk",  # Different user!
            session_id="session-2",  # Different session!
            metadata={
                "tenant_id": "tenant-2",  # Different tenant!
            },
        )
        print(f"Response: {response2.content}\n")

        print("=" * 60)
        print("Success! Check your MCP server logs to see the headers.")
        print("=" * 60)

    finally:
        # Step 6: Clean up
        await mcp_tools.close()
        print("\nConnection closed")


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    asyncio.run(main())
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/mcp/dynamic_headers/client.py`

## 概述

Agent with MCP tools using Dynamic Headers

本示例归类：**单 Agent**；模型相关类型：`OpenAIChat`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | 'agent-1' | `Agent(...)` |
| `model` | OpenAIChat(id='gpt-5.2'…) | `Agent(...)` |
| `markdown` | False | `Agent(...)` |
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
