# retry.md — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example demonstrating how to set up retries with llama.cpp."""

from agno.agent import Agent
from agno.models.llama_cpp import LlamaCpp

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

# We will use a deliberately wrong model ID, to trigger retries.
wrong_model_id = "llama-cpp-wrong-id"

agent = Agent(
    model=LlamaCpp(
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

> 源文件：`cookbook/90_models/llama_cpp/retry.py`

## 概述

**`LlamaCpp` 错误 id + 重试**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LlamaCpp(id="llama-cpp-wrong-id", retries=3, delay_between_retries=1, exponential_backoff=True)` | LlamaCpp |

## Mermaid 流程图

```mermaid
flowchart TD
    A["本地 server 失败"] --> B["【关键】重试"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/llama_cpp/llama_cpp.py` | `LlamaCpp` |
