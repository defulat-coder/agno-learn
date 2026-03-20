# basic.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Ibm Basic
=========

Cookbook example for `ibm/watsonx/basic.py`.
"""

from agno.agent import Agent, RunOutput  # noqa
from agno.models.ibm import WatsonX
import asyncio

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=WatsonX(id="mistralai/mistral-small-3-1-24b-instruct-2503"), markdown=True
)

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

> 源文件：`cookbook/90_models/ibm/watsonx/basic.py`

## 概述

本示例展示 **`WatsonX`** 的同步/流式/异步调用，`markdown=True`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `WatsonX(id="mistralai/mistral-small-3-1-24b-instruct-2503")` | WatsonX |
| `markdown` | `True` | Markdown 附加段 |

## System Prompt 组装

静态段含：

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

用户消息：`Share a 2 sentence horror story`

## 完整 API 请求

见 `watsonx.py`：`client.chat(messages=formatted_messages, **request_params)`。

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/ibm/watsonx.py` | `invoke` |
