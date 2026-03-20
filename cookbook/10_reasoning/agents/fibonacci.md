# fibonacci.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Fibonacci Script Planning
=========================

Demonstrates built-in and DeepSeek-backed reasoning for coding guidance.
"""

from agno.agent import Agent
from agno.models.deepseek import DeepSeek
from agno.models.openai import OpenAIChat

# ---------------------------------------------------------------------------
# Create Agents
# ---------------------------------------------------------------------------
task = "Give me steps to write a python script for fibonacci series"

cot_agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    reasoning=True,
    markdown=True,
)

deepseek_agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    reasoning_model=DeepSeek(id="deepseek-reasoner"),
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agents
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    print("=== Built-in Chain Of Thought ===")
    cot_agent.print_response(task, stream=True, show_full_reasoning=True)

    print("\n=== DeepSeek Reasoning Model ===")
    deepseek_agent.print_response(task, stream=True, show_full_reasoning=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/10_reasoning/agents/fibonacci.py`

## 概述

与 `analyse_treaty_of_versailles.py` 同构，任务改为 **Fibonacci 脚本步骤**；对比内置推理与 DeepSeek `reasoning_model`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `task` | Fibonacci 脚本步骤 | 编程向 |

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/deepseek` | 推理模型 |
