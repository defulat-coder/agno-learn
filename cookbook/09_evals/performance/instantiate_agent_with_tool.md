# instantiate_agent_with_tool.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Agent-with-Tool Instantiation Performance Evaluation
====================================================

Demonstrates measuring instantiation performance for a tooled agent.
"""

from typing import Literal

from agno.agent import Agent
from agno.eval.performance import PerformanceEval
from agno.models.openai import OpenAIChat


# ---------------------------------------------------------------------------
# Create Benchmark Tool
# ---------------------------------------------------------------------------
def get_weather(city: Literal["nyc", "sf"]):
    """Use this to get weather information."""
    if city == "nyc":
        return "It might be cloudy in nyc"
    elif city == "sf":
        return "It's always sunny in sf"


tools = [get_weather]


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
def instantiate_agent():
    return Agent(model=OpenAIChat(id="gpt-4o"), tools=tools)  # type: ignore


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
instantiation_perf = PerformanceEval(
    name="Agent Instantiation", func=instantiate_agent, num_iterations=1000
)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    instantiation_perf.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/instantiate_agent_with_tool.py`

## 概述

本示例测量 **带工具列表的 Agent 实例化**：`Agent(model=gpt-4o, tools=[get_weather])`，`num_iterations=1000`，不发起网络请求。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `tools` | `get_weather` 函数 | 天气占位工具 |

## 核心组件解析

工具注册与 schema 生成会增加构造时间，对比纯 `instantiate_agent.py`。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/agent.py` | 工具解析 |
