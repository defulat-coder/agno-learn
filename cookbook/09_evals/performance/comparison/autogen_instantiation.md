# autogen_instantiation.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
AutoGen Instantiation Performance Evaluation
============================================

Demonstrates agent instantiation benchmarking with AutoGen.
"""

from typing import Literal

from agno.eval.performance import PerformanceEval
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient


# ---------------------------------------------------------------------------
# Create Benchmark Tool
# ---------------------------------------------------------------------------
def get_weather(city: Literal["nyc", "sf"]):
    """Use this to get weather information."""
    if city == "nyc":
        return "It might be cloudy in nyc"
    elif city == "sf":
        return "It's always sunny in sf"
    else:
        raise AssertionError("Unknown city")


tools = [get_weather]


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
def instantiate_agent():
    return AssistantAgent(
        name="assistant",
        model_client=OpenAIChatCompletionClient(
            model="gpt-4o",
            model_info={
                "vision": False,
                "function_calling": True,
                "json_output": False,
                "family": "gpt-4o",
                "structured_output": True,
            },
        ),
        tools=tools,
    )


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
autogen_instantiation = PerformanceEval(func=instantiate_agent, num_iterations=1000)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    autogen_instantiation.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/comparison/autogen_instantiation.py`

## 概述

本示例用 **`PerformanceEval`** 包裹 **AutoGen `AssistantAgent` 实例化**（`autogen_agentchat` + `OpenAIChatCompletionClient`），对比 Agno 自身实例化基准。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| 依赖 | `autogen_agentchat`、`autogen_ext` | 第三方 |
| `func` | `instantiate_agent` → `AssistantAgent(...)` | 仅构造 |

## 核心组件解析

不测 Agno `Agent`，测竞品框架冷启动成本；`num_iterations` 见文件末尾。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | 通用计时器 |
