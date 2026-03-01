# team_response_with_memory_and_reasoning.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/team_response_with_memory_and_reasoning.py`

## 概述

本示例展示 **`PerformanceEval`** 对**带推理工具（ReasoningTools）的复杂 Team** 进行内存增长测量：Team + 2 个成员均使用 `OpenAIResponses`（Responses API）+ `ReasoningTools`，每用户 4 次连续会话请求，5 并发用户。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `func` | `run_team`（async） | 被测函数 |
| `num_iterations` | `5` | 5 次迭代 |
| `measure_runtime` | `False` | 只测内存 |
| `memory_growth_tracking` | `True` | 内存增长追踪 |
| `top_n_memory_allocations` | `10` | Top10 分配 |
| **模型** | `OpenAIResponses("gpt-4o")` | Responses API |

## 核心组件解析

### OpenAIResponses vs OpenAIChat

本文件所有 Agent/Team 使用 `OpenAIResponses`（Responses API），与其他示例的 `OpenAIChat`（Chat Completions API）不同：

| 特性 | OpenAIResponses | OpenAIChat |
|------|----------------|-----------|
| API 端点 | `client.responses.create()` | `client.chat.completions.create()` |
| 角色映射 | `system → developer` | `system → system` |
| 参数 | `input=[...]` | `messages=[...]` |

### ReasoningTools + add_instructions=True

```python
tools=[ReasoningTools(add_instructions=True), get_weather]
# → ReasoningTools 会向 system prompt 注入推理思考链指引
```

### 每用户 4 次串行请求（跨会话历史）

```python
async def run_team_for_user(user: str, ...):
    session_id = f"session_{user}_{uuid.uuid4()}"
    # 4 次请求共享同一 session_id，形成连续对话历史
    _ = team.arun(input=f"I love {random_city}!", user_id=user, session_id=session_id)
    _ = team.arun(input=f"Create a report on activities and weather in {random_city}.", ...)
    _ = team.arun(input=f"What else can you tell me about {random_city}?", ...)
    _ = team.arun(input=f"What other cities are similar to {random_city}?", ...)
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `_async_measure_memory_with_growth_tracking()` L437 | 内存追踪 |
| `agno/models/openai/responses.py` | `OpenAIResponses` L31 | Responses API 模型 |
| `agno/tools/reasoning.py` | `ReasoningTools` L10 | 推理工具 |
