# ibm_watsonx_default_COT.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
WatsonX Default COT Fallback
============================

Demonstrates default chain-of-thought behavior with an IBM WatsonX model.
"""

from agno.agent import Agent
from agno.models.ibm import WatsonX

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
reasoning_agent = Agent(
    model=WatsonX(id="meta-llama/llama-3-3-70b-instruct"),
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

> 源文件：`cookbook/10_reasoning/agents/ibm_watsonx_default_COT.py`

## 概述

本示例展示 **`WatsonX(id="meta-llama/llama-3-3-70b-instruct")` + `reasoning=True`** 的默认 COT，`debug_mode=True`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `WatsonX` | IBM 托管 |

## 完整 API 请求

见 `agno/models/ibm` 中 WatsonX 的 `invoke`/`ainvoke`。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/ibm` | WatsonX |
