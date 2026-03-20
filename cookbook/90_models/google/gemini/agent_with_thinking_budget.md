# agent_with_thinking_budget.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
An example of how to use the thinking budget parameter with the Gemini model.
This requires `google-genai > 1.10.0`

- Turn off thinking use thinking_budget=0
- Turn on dynamic thinking use thinking_budget=-1
- To use a specific thinking token budget (e.g. 1280) use thinking_budget=1280
- Use include_thoughts=True to get the thought summaries in the response.
"""

from agno.agent import Agent
from agno.models.google import Gemini

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

task = (
    "Three missionaries and three cannibals need to cross a river. "
    "They have a boat that can carry up to two people at a time. "
    "If, at any time, the cannibals outnumber the missionaries on either side of the river, the cannibals will eat the missionaries. "
    "How can all six people get across the river safely? Provide a step-by-step solution and show the solutions as an ascii diagram"
)

agent = Agent(
    model=Gemini(id="gemini-2.5-pro", thinking_budget=1280, include_thoughts=True),
    markdown=True,
)
agent.print_response(task, stream=True)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/google/gemini/agent_with_thinking_budget.py`

## 概述

本示例展示 **Gemini `thinking_budget` + `include_thoughts`**（需 `google-genai > 1.10.0`）：`gemini-2.5-pro` 上设置 `thinking_budget=1280`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-2.5-pro", thinking_budget=1280, include_thoughts=True)` | 见 `libs/agno/agno/models/google/gemini.py` 请求参数 |
| `markdown` | `True` | |

## 完整 API 请求

```python
# gemini.py L527-531
get_client().models.generate_content(
    model="gemini-2.5-pro",
    contents=formatted_messages,
    **request_kwargs,  # 含 thinking 相关配置
)
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["推理题 task"] --> B["【关键】thinking_budget / include_thoughts"]
    B --> C["generate_content stream"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `invoke()` / `get_request_params` | thinking 参数 |
| `agno/agent/_messages.py` | `get_system_message()` | system 文本 |
