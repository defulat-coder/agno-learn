# crewai_instantiation.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
CrewAI Instantiation Performance Evaluation
===========================================

Demonstrates agent instantiation benchmarking with CrewAI.
"""

from typing import Literal

from agno.eval.performance import PerformanceEval
from crewai.agent import Agent
from crewai.tools import tool


# ---------------------------------------------------------------------------
# Create Benchmark Tool
# ---------------------------------------------------------------------------
@tool("Tool Name")
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
    return Agent(
        llm="gpt-4o",
        role="Test Agent",
        goal="Be concise, reply with one sentence.",
        tools=tools,
        backstory="Test",
    )


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
crew_instantiation = PerformanceEval(func=instantiate_agent, num_iterations=1000)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    crew_instantiation.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/comparison/crewai_instantiation.py`

## 概述

本示例用 **`PerformanceEval`** 测量 **CrewAI** 框架下 Agent/Crew 的实例化耗时，用于与 Agno 对照（具体类名与 `func` 以源文件为准）。

## 核心组件解析

不包含 Agno `get_system_message`；属于跨框架微基准。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | `PerformanceEval` |
