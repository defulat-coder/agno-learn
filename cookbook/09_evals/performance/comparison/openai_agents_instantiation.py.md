# openai_agents_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/openai_agents_instantiation.py`

## 概述

本示例展示用 **`PerformanceEval`** 对 **OpenAI Agents SDK `Agent`** 的实例化性能测试，迭代 1000 次，测量含 `function_tool()` 包装工具的构造开销。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `func` | `instantiate_agent`（OpenAI Agents SDK） | 被测函数 |
| `num_iterations` | `1000` | 高频迭代 |

## 核心组件解析

### 被测函数（OpenAI Agents SDK）

```python
from agents import Agent, function_tool

def instantiate_agent():
    return Agent(
        name="Haiku agent",
        instructions="Always respond in haiku form",
        model="o3-mini",
        tools=[function_tool(get_weather)],  # function_tool 包装裸函数
    )
```

`function_tool()` 将裸函数包装为 OpenAI Agents SDK 的工具格式（类似 Agno 的裸函数工具注册，但实现不同）。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `PerformanceEval.run()` L481 | 测量主流程 |
