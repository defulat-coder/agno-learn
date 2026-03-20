# mcp_tools_existing_lifespan.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Example AgentOS app where the agent has MCPTools.

AgentOS handles the lifespan of the MCPTools internally.

In addition you can pass your own lifespan to AgentOS.
"""

from contextlib import asynccontextmanager

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.models.anthropic import Claude
from agno.os import AgentOS
from agno.tools.mcp import MCPTools
from agno.utils.log import log_info

# ---------------------------------------------------------------------------
# Create Example
# ---------------------------------------------------------------------------

# Setup the database
db = SqliteDb(db_file="tmp/agentos.db")

mcp_tools = MCPTools(transport="streamable-http", url="https://docs.agno.com/mcp")

# Setup basic support agent
agno_support_agent = Agent(
    id="agno-support-agent",
    name="Agno Support Agent",
    model=Claude(id="claude-sonnet-4-0"),
    db=db,
    tools=[mcp_tools],
    add_history_to_context=True,
    num_history_runs=3,
    markdown=True,
)


@asynccontextmanager
async def lifespan(app):
    log_info("Starting My FastAPI App")
    yield
    log_info("Stopping My FastAPI App")


agent_os = AgentOS(
    description="Example app with MCP Tools",
    agents=[agno_support_agent],
    enable_mcp_server=True,
    lifespan=lifespan,
)


app = agent_os.get_app()

# ---------------------------------------------------------------------------
# Run Example
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    """Run your AgentOS.

    You can see test your AgentOS at:
    http://localhost:7777/docs

    """
    # Don't use reload=True here, this can cause issues with the lifespan
    agent_os.serve(app="mcp_tools_existing_lifespan:app")
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/05_agent_os/mcp_demo/mcp_tools_existing_lifespan.py`

## 概述

本示例展示 Agno 的 **自定义 FastAPI `lifespan` + `enable_mcp_server=True` + MCPTools** 机制：在 `AgentOS` 上同时传入 `lifespan`（启动/停止日志）与内置 MCP 端点，演示 **与框架默认 lifespan 合成** 时的写法（避免与 `mcp_tools_example` 仅 Agent 的差异）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `lifespan` | `asynccontextmanager` 打印日志 | 自定义 |
| `enable_mcp_server` | `True` | 内置 `/mcp` |
| `mcp_tools` | 同 `mcp_tools_example` | 远程文档 MCP |
| `agno_support_agent` | `Claude` + `mcp_tools` |  |

## 运行机制与因果链

`lifespan` 在 app 启动时 `yield` 前后执行；MCP 工具 lifespan 仍由 AgentOS 协调（注释）。

## System Prompt 组装

同 `mcp_tools_example.py`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["FastAPI lifespan"] --> B["【关键】AgentOS 合成"]
    B --> C["MCPTools + /mcp"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/os` | `AgentOS(lifespan=..., enable_mcp_server=...)` | 生命周期 |
