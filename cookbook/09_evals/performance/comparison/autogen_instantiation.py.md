# autogen_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/autogen_instantiation.py`

## 概述

本示例展示用 **`PerformanceEval`** 对 **AutoGen `AssistantAgent`** 的实例化性能进行基准测试，与 Agno Agent 进行横向对比，迭代 1000 次统计实例化耗时和内存。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `func` | `instantiate_agent`（AutoGen） | 被测函数 |
| `num_iterations` | `1000` | 高频迭代 |
| `warmup_runs` | `10`（默认） | 预热 10 次 |
| `name` | `None` | 未设置 |

## 核心组件解析

### 被测函数（AutoGen AssistantAgent）

```python
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
```

### 对比方案

| 文件 | 框架 | 被测对象 |
|------|------|---------|
| `instantiate_agent_with_tool.py` | **Agno** | `Agent(model, tools)` |
| `autogen_instantiation.py` | **AutoGen** | `AssistantAgent(name, model_client, tools)` |
| `crewai_instantiation.py` | **CrewAI** | `Agent(llm, role, tools)` |
| `langgraph_instantiation.py` | **LangGraph** | `create_react_agent(model, tools)` |
| `openai_agents_instantiation.py` | **OpenAI Agents** | `Agent(name, tools)` |
| `smolagents_instantiation.py` | **Smolagents** | `ToolCallingAgent(tools, model)` |
| `pydantic_ai_instantiation.py` | **PydanticAI** | `Agent(model)` + `@agent.tool_plain` |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `PerformanceEval.run()` L481 | 测量主流程 |
| `agno/eval/performance.py` | `PerformanceResult` L19 | 统计结果 |
