# retry.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example demonstrating how to set up retries with AWS Bedrock."""

from agno.agent import Agent
from agno.models.aws import AwsBedrock

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

# We will use a deliberately wrong model ID, to trigger retries.
wrong_model_id = "aws-bedrock-wrong-id"

agent = Agent(
    model=AwsBedrock(
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

> 源文件：`cookbook/90_models/aws/retry.py`

## 概述

**AwsBedrock** 故意错误 `id` 与 **retries / delay / exponential_backoff**，演示模型层重试。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AwsBedrock(id="aws-bedrock-wrong-id", retries=3, ...)` | 重试 |

## 运行机制与因果链

与 `anthropic/retry.py` 相同意图，适配器为 **Converse**（`bedrock.py`）。

## System Prompt 组装

默认极短；运行时打印。

## Mermaid 流程图

```mermaid
flowchart TD
    A["错误 modelId"] --> B["【关键】重试后仍失败"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/aws/bedrock.py` | `invoke` | Converse |
