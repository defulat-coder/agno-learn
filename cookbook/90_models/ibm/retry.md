# retry.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example demonstrating how to set up retries with IBM WatsonX."""

from agno.agent import Agent
from agno.models.ibm import WatsonX

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

# We will use a deliberately wrong model ID, to trigger retries.
wrong_model_id = "watsonx-wrong-id"

agent = Agent(
    model=WatsonX(
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

> 源文件：`cookbook/90_models/ibm/retry.py`

## 概述

本示例演示 **`WatsonX` 模型 id 故意错误** 时 **`retries` / `delay_between_retries` / `exponential_backoff`** 行为。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `WatsonX(id="watsonx-wrong-id", retries=3, delay_between_retries=1, exponential_backoff=True)` | IBM WatsonX |

## 完整 API 请求

`WatsonX.invoke`（`agno/models/ibm/watsonx.py` 约 L162+）使用 `client.chat(messages=..., **request_params)`，非 OpenAI 字面量 `chat.completions`，而是 WatsonX SDK 封装。

## Mermaid 流程图

```mermaid
flowchart TD
    A["WatsonX.chat"] --> B["【关键】失败重试"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/ibm/watsonx.py` | `invoke` L162+ |
