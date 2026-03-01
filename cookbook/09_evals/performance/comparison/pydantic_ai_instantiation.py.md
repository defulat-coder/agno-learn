# pydantic_ai_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/pydantic_ai_instantiation.py`

## 概述

本示例展示用 **`PerformanceEval`** 对 **PydanticAI `Agent`** 的实例化性能测试：PydanticAI 使用 `@agent.tool_plain` 装饰器在 Agent 构造后注册工具，这意味着工具注册是实例化开销的一部分，迭代 1000 次。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `func` | `instantiate_agent`（PydanticAI） | 被测函数 |
| `num_iterations` | `1000` | 高频迭代 |

## 核心组件解析

### 被测函数（PydanticAI，工具在函数内注册）

```python
def instantiate_agent():
    agent = Agent("openai:gpt-4o", system_prompt="Be concise, reply with one sentence.")

    @agent.tool_plain  # ← 工具注册在 instantiate_agent() 内部
    def get_weather(city: Literal["nyc", "sf"]):
        """Use this to get weather information."""
        ...

    return agent
```

注意：PydanticAI 的工具通过装饰器注册，工具定义必须在每次 `instantiate_agent()` 调用内执行，这与其他框架（预定义工具列表）不同，可能影响实例化性能。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `PerformanceEval.run()` L481 | 测量主流程 |
