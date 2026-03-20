# smolagents_instantiation.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Smolagents Instantiation Performance Evaluation
===============================================

Demonstrates agent instantiation benchmarking with Smolagents.
"""

from agno.eval.performance import PerformanceEval
from smolagents import InferenceClientModel, Tool, ToolCallingAgent


# ---------------------------------------------------------------------------
# Create Benchmark Tool
# ---------------------------------------------------------------------------
class WeatherTool(Tool):
    name = "weather_tool"
    description = """
    This is a tool that tells the weather"""
    inputs = {
        "city": {
            "type": "string",
            "description": "The city to look up",
        }
    }
    output_type = "string"

    def forward(self, city: str):
        """Use this to get weather information."""
        if city == "nyc":
            return "It might be cloudy in nyc"
        elif city == "sf":
            return "It's always sunny in sf"
        else:
            raise AssertionError("Unknown city")


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
def instantiate_agent():
    return ToolCallingAgent(
        tools=[WeatherTool()],
        model=InferenceClientModel(model_id="meta-llama/Llama-3.3-70B-Instruct"),
    )


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
smolagents_instantiation = PerformanceEval(func=instantiate_agent, num_iterations=1000)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    smolagents_instantiation.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/comparison/smolagents_instantiation.py`

## 概述

本示例测量 **smolagents** 库中 Agent 构造耗时，纳入 `PerformanceEval` 对比实验（实现以源文件为准）。

## 核心组件解析

用于与 Agno `Agent(...)` 冷启动对比。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | 性能评估 |
