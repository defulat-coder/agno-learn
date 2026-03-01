# crewai_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/crewai_instantiation.py`

## 概述

本示例展示用 **`PerformanceEval`** 对 **CrewAI `Agent`** 的实例化性能进行基准测试，测量含 `@tool` 装饰器工具的 CrewAI Agent 构造开销，迭代 1000 次。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `func` | `instantiate_agent`（CrewAI） | 被测函数 |
| `num_iterations` | `1000` | 高频迭代 |

## 核心组件解析

### 被测函数（CrewAI）

```python
@tool("Tool Name")
def get_weather(city: Literal["nyc", "sf"]):
    """Use this to get weather information."""
    ...

def instantiate_agent():
    return Agent(
        llm="gpt-4o",
        role="Test Agent",
        goal="Be concise, reply with one sentence.",
        tools=tools,
        backstory="Test",
    )
```

CrewAI 的 `@tool` 装饰器相比 Agno 的裸函数工具，有额外的装饰器解析开销，这会影响实例化时间。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `PerformanceEval.run()` L481 | 测量主流程 |
