# tool_use.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Nebius Tool Use
===============

Cookbook example for `nebius/tool_use.py`.
"""

import asyncio

from agno.agent import Agent
from agno.models.nebius import Nebius
from agno.tools.websearch import WebSearchTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=Nebius(id="Qwen/Qwen3-30B-A3B"),
    tools=[WebSearchTools()],
    markdown=True,
)

# Print the response in the terminal

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    # --- Sync ---
    agent.print_response("Whats happening in France?")

    # --- Sync + Streaming ---
    agent.print_response("Whats happening in France?", stream=True)

    # --- Async ---
    asyncio.run(agent.aprint_response("Whats happening in France?"))

    # --- Async + Streaming ---
    asyncio.run(agent.aprint_response("Whats happening in France?", stream=True))
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/nebius/tool_use.py`

## 概述

本示例展示 **`Nebius(id="Qwen/Qwen3-30B-A3B")` + WebSearchTools**，同步/异步与流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nebius(id="Qwen/Qwen3-30B-A3B")` | Chat |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | 默认 |

用户消息：`"Whats happening in France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["print_response"] --> B["【关键】工具调用"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nebius/nebius.py` | `Nebius` |
