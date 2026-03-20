# message_history_in_hooks.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Message History In Hooks
=============================

Access the current run's message history inside tool pre/post hooks
via run_context.messages.
"""

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.run.base import RunContext
from agno.tools import FunctionCall, tool

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


def pre_hook(run_context: RunContext, fc: FunctionCall):
    msgs = run_context.messages
    count = len(msgs) if msgs else 0
    print(f"[pre-hook] {fc.function.name} - {count} messages in run")


def post_hook(run_context: RunContext, fc: FunctionCall):
    msgs = run_context.messages
    count = len(msgs) if msgs else 0
    print(
        f"[post-hook] {fc.function.name} returned '{fc.result}' - {count} messages in run"
    )


@tool(pre_hook=pre_hook, post_hook=post_hook)
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"Sunny, 72F in {city}"


agent = Agent(
    model=OpenAIChat(id="gpt-4o-mini"),
    tools=[get_weather],
    instructions=["Use the tools to help the user."],
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    agent.print_response("What is the weather in San Francisco?")
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/tool_hooks/message_history_in_hooks.py`

## 概述

本示例展示 **`@tool(pre_hook=..., post_hook=...)`** 中钩子接收 **`RunContext`**，通过 **`run_context.messages`** 读取 **当前 run 内消息条数**，用于调试/观测。

**核心配置一览**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o-mini")` |  |
| `tools` | `[get_weather]` |  |
| `instructions` | `["Use the tools to help the user."]` |  |

## 运行机制与因果链

pre/post 在工具执行前后打印消息计数；**不**改变工具返回值逻辑。

## Mermaid 流程图

```mermaid
flowchart TD
    A["RunContext.messages"] --> B["【关键】钩子里看历史长度"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/run/base.py` | `RunContext` |
