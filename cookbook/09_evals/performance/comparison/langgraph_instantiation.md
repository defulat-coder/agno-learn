# langgraph_instantiation.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
LangGraph Instantiation Performance Evaluation
==============================================

Demonstrates agent instantiation benchmarking with LangGraph.
"""

from typing import Literal

from agno.eval.performance import PerformanceEval
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent


# ---------------------------------------------------------------------------
# Create Benchmark Tool
# ---------------------------------------------------------------------------
@tool
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
    return create_react_agent(model=ChatOpenAI(model="gpt-4o"), tools=tools)


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
langgraph_instantiation = PerformanceEval(func=instantiate_agent, num_iterations=1000)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    langgraph_instantiation.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/comparison/langgraph_instantiation.py`

## 概述

本示例用 **`PerformanceEval`** 测量 **LangGraph** 图或节点构建/编译的实例化耗时（具体 API 以源文件为准），用于横向对比。

## 核心组件解析

纯框架构造路径，不涉及 Agno Agent。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | 计时封装 |
