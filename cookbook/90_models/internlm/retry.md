# retry.md — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example demonstrating how to set up retries with InternLM."""

from agno.agent import Agent
from agno.models.internlm import InternLM

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

# We will use a deliberately wrong model ID, to trigger retries.
wrong_model_id = "internlm-wrong-id"

agent = Agent(
    model=InternLM(
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

> 源文件：`cookbook/90_models/internlm/retry.py`

## 概述

**`InternLM` 错误 id + 重试参数**，与其它厂商 `retry.py` 模式一致。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `InternLM(id="internlm-wrong-id", retries=3, delay_between_retries=1, exponential_backoff=True)` | InternLM |

## 完整 API 请求

以 `InternLM` 适配器 `invoke` 为准（通常为 OpenAI 兼容或厂商 SDK）。

## Mermaid 流程图

```mermaid
flowchart TD
    A["invoke 失败"] --> B["【关键】重试"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/internlm/` | `InternLM` 类 |
