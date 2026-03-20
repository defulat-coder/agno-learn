# retry.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example demonstrating how to set up retries with NVIDIA."""

from agno.agent import Agent
from agno.models.nvidia import Nvidia

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

# We will use a deliberately wrong model ID, to trigger retries.
wrong_model_id = "nvidia-wrong-id"

agent = Agent(
    model=Nvidia(
        id=wrong_model_id,
        retries=3,  # Number of times to retry the request.
        delay_between_retries=1,  # Delay between retries in seconds.
        exponential_backoff=True,  # If True, the delay between retries is doubled each time.
    ),
)

agent.print_response("What is the capital of France?")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/nvidia/retry.py`

## 概述

本示例展示 **Nvidia 模型重试**与 `nvidia-wrong-id`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nvidia(id=wrong_model_id, retries=3, delay_between_retries=1, exponential_backoff=True)` | 重试 |

用户消息：`"What is the capital of France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["invoke"] --> B["【关键】重试"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nvidia/` | `Nvidia` |
