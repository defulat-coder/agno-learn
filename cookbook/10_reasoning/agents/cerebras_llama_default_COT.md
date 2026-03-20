# cerebras_llama_default_COT.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Cerebras Default COT Fallback
=============================

Demonstrates default chain-of-thought behavior with a Cerebras model.
"""

from agno.agent import Agent
from agno.models.cerebras import Cerebras

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
reasoning_agent = Agent(
    model=Cerebras(id="llama-3.3-70b"),
    reasoning=True,
    debug_mode=True,
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    reasoning_agent.print_response(
        "Give me steps to write a python script for fibonacci series",
        stream=True,
        show_full_reasoning=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/10_reasoning/agents/cerebras_llama_default_COT.py`

## 概述

本示例展示 **`Cerebras(id="llama-3.3-70b")` + `reasoning=True` + `debug_mode=True`** 的默认 COT 回退行为，任务为 Fibonacci 步骤。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cerebras` | 非 OpenAI |
| `reasoning` | `True` | 内置推理 |

## 完整 API 请求

以 `agno/models/cerebras` 中客户端为准（非 `chat.completions` 若厂商不同）。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/cerebras` | Cerebras 适配 |
