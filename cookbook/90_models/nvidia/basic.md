# basic.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Nvidia Basic
============

Cookbook example for `nvidia/basic.py`.
"""

from agno.agent import Agent, RunOutput  # noqa
from agno.models.nvidia import Nvidia
import asyncio

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(model=Nvidia(id="meta/llama-3.3-70b-instruct"), markdown=True)

# Get the response in a variable
# run: RunOutput = agent.run("Share a 2 sentence horror story")
# print(run.content)

# Print the response in the terminal

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    # --- Sync ---
    agent.print_response("Share a 2 sentence horror story")

    # --- Sync + Streaming ---
    agent.print_response("Share a 2 sentence horror story", stream=True)

    # --- Async ---
    asyncio.run(agent.aprint_response("Share a 2 sentence horror story"))

    # --- Async + Streaming ---
    asyncio.run(agent.aprint_response("Share a 2 sentence horror story", stream=True))
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/nvidia/basic.py`

## 概述

本示例展示 **`Nvidia(id="meta/llama-3.3-70b-instruct")`**（NVIDIA NIM/OpenAI 兼容）基础对话。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nvidia(id="meta/llama-3.3-70b-instruct")` | OpenAILike |
| `markdown` | `True` | 默认 |

用户消息：`"Share a 2 sentence horror story"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["Nvidia 端点"] --> B["【关键】Chat Completions"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nvidia/` | `Nvidia` |
