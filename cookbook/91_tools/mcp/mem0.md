# mem0.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

````python
"""
‍ Mem0 MCP - Personalized Code Reviewer

This example demonstrates how to use Agno's MCP integration together with Mem0, to build a personalized code reviewer.

- Run your Mem0 MCP server. Full instructions: https://github.com/mem0ai/mem0-mcp
- Run: `uv pip install agno mcp` to install the dependencies
"""

import asyncio
from textwrap import dedent

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.mcp import MCPTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


mcp_server_url = "http://localhost:8080/sse"


async def run_agent(message: str) -> None:
    async with MCPTools(url=mcp_server_url, transport="sse") as mcp_tools:
        agent = Agent(
            tools=[mcp_tools],
            model=OpenAIChat(id="o4-mini"),
            instructions=dedent(
                """
                You are a professional code reviewer. You help users keep their code clean and on line with their preferences.
                You have access to some tools to keep track of coding preferences you need to enforce when reviewing code.
                You will be given a code snippet and you need to review it and provide feedback on it.
                """
            ),
        )
        await agent.aprint_response(message, stream=True)


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    # The agent will use mem0 memory to keep track of the user's preferences.
    asyncio.run(
        run_agent(
            "When possible, use the walrus operator to make the code more readable."
        )
    )
    # The agent will review your code and propose improvements based on your preferences.
    asyncio.run(
        run_agent(
            dedent(
                """
Please, review this Python snippet:

```python
def process_data(data):
    length = len(data)
    if length > 10:
        print(f"Processing {length} items")
        return data[:10]
    return data

# Example usage
items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
result = process_data(items)
```
"""
            )
        )
    )
````

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/mcp/mem0.py`

## 概述

‍ Mem0 MCP - Personalized Code Reviewer

本示例归类：**单 Agent**；模型相关类型：`OpenAIChat`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | OpenAIChat(id='o4-mini'…) | `Agent(...)` |
| `instructions` | dedent('\n                You are a professional code reviewer. You help users keep their code clean and on line with their preferences.\n                You have access to some tools to keep track of coding preferences you need to enforce when reviewing code.\n                You will be given a code snippet and you need to review it and provide feedback on it.\n                '…) | `Agent(...)` |
| （Model 类） | `OpenAIChat` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ mem0.py              │  ──▶  │ Agent → get_run_messages → Model │
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
