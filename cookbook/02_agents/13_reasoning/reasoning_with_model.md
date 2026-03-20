# reasoning_with_model.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Reasoning With Model
=============================

Use a separate reasoning model with configurable step limits.
"""

from agno.agent import Agent
from agno.models.openai import OpenAIResponses

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
agent = Agent(
    model=OpenAIResponses(id="gpt-5.2"),
    # Use a separate model for the reasoning/thinking step
    reasoning_model=OpenAIResponses(id="gpt-5-mini"),
    reasoning=True,
    reasoning_min_steps=2,
    reasoning_max_steps=5,
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    agent.print_response(
        "A farmer has 17 sheep. All but 9 die. How many sheep are left?",
        stream=True,
        show_full_reasoning=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/02_agents/13_reasoning/reasoning_with_model.py`

## 概述

本示例展示 **主模型与 `reasoning_model` 分离**：主模型 `gpt-5.2` 负责最终回答，`reasoning_model=gpt-5-mini` 承担思考步骤；同样 `reasoning_min/max_steps` 与 `show_full_reasoning`。

**核心配置一览：**

| 配置项 | 值 |
|--------|-----|
| `model` | `OpenAIResponses(id="gpt-5.2")` |
| `reasoning_model` | `OpenAIResponses(id="gpt-5-mini")` |
| `reasoning` | `True` |
| `markdown` | `True` |

## 运行机制与因果链

成本/延迟拆分：小模型做推理草稿，大模型润色输出（具体调度见 `_run` 推理分支）。

参照用户句：羊只数量谜题（`.py` 字面量）。

## Mermaid 流程图

```mermaid
flowchart TD
    RM["reasoning_model 思考"] --> M["【关键】主 model 定稿"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/agent.py` | `reasoning_model` 属性 |
