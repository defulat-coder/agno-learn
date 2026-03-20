# retry.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example demonstrating how to set up retries with Nebius."""

from agno.agent import Agent
from agno.models.nebius import Nebius

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

# We will use a deliberately wrong model ID, to trigger retries.
wrong_model_id = "nebius-wrong-id"

agent = Agent(
    model=Nebius(
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

> 源文件：`cookbook/90_models/nebius/retry.py`

## 概述

本示例展示 **Nebius 模型重试参数**与错误 `nebius-wrong-id`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nebius(id=wrong_model_id, retries=3, delay_between_retries=1, exponential_backoff=True)` | 重试 |

用户消息：`"What is the capital of France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["invoke 失败"] --> B["【关键】退避重试"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nebius/nebius.py` | `Nebius` |
