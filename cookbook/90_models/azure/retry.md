# retry.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example demonstrating how to set up retries with Azure AI Foundry."""

from agno.agent import Agent
from agno.models.azure import AzureAIFoundry

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

# We will use a deliberately wrong model ID, to trigger retries.
wrong_model_id = "azure-wrong-id"

agent = Agent(
    model=AzureAIFoundry(
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

> 源文件：`cookbook/90_models/azure/retry.py`

## 概述

**AzureAIFoundry** 错误 `id` + **retries** 配置，演示 Foundry 路径上的重试。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureAIFoundry(id="azure-wrong-id", retries=3, ...)` | 重试 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["HttpResponseError"] --> B["【关键】重试"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/ai_foundry.py` | `invoke` L228+ | 异常映射 |
