# langgraph_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/langgraph_instantiation.py`

## 概述

本示例展示用 **`PerformanceEval`** 对 **LangGraph `create_react_agent`** 的实例化性能测试，测量含 `@tool` 装饰器的 LangGraph ReAct Agent 构造开销，迭代 1000 次。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `func` | `instantiate_agent`（LangGraph） | 被测函数 |
| `num_iterations` | `1000` | 高频迭代 |

## 核心组件解析

### 被测函数（LangGraph）

```python
@tool
def get_weather(city: Literal["nyc", "sf"]):
    """Use this to get weather information."""
    ...

def instantiate_agent():
    return create_react_agent(model=ChatOpenAI(model="gpt-4o"), tools=tools)
```

`create_react_agent` 会构建完整的 LangGraph StateGraph（节点 + 边），比单纯的 Agent 构造更重量级。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `PerformanceEval.run()` L481 | 测量主流程 |
