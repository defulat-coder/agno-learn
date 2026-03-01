# state_in_function.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/session_state/state_in_function.py`

## 概述

本示例展示 Agno Workflow **函数执行器通过 `session_state` 参数直接读写状态**的两种模式：同步函数（返回 `StepOutput`）和流式函数（`yield` `StepOutput`），都可以通过函数签名中的 `session_state: dict` 参数接收并修改工作流会话状态，实现跨 run 的计划计数和历史累积。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| executor 签名 | `fn(step_input, session_state: dict)` | session_state 自动注入 |
| 修改方式 | 直接修改字典 `session_state["key"] = value` | 原位修改，持久化 |
| 流式 executor | `yield event` + `yield StepOutput(...)` | 支持 streaming |

## 核心组件解析

### 同步 executor 使用 session_state

```python
def custom_content_planning_function(
    step_input: StepInput,
    session_state: dict,            # Workflow 自动注入
) -> StepOutput:
    # 读取并更新计数器
    session_state.setdefault("content_plans", [])
    session_state.setdefault("plan_counter", 0)
    session_state["plan_counter"] += 1

    # 内部调用 Agent
    response = content_planner.run(planning_prompt)

    # 存储结果到 session_state
    session_state["content_plans"].append({
        "id": session_state["plan_counter"],
        "topic": step_input.input,
        "content": response.content,
    })

    return StepOutput(content=enhanced_content)
```

### 流式 executor 使用 session_state

```python
from agno.run.workflow import WorkflowRunOutputEvent
from typing import Iterator, Union

def custom_content_planning_function_stream(
    step_input: StepInput,
    session_state: dict,
) -> Iterator[Union[WorkflowRunOutputEvent, StepOutput]]:
    session_state["plan_counter"] = session_state.get("plan_counter", 0) + 1

    # 流式 Agent 调用
    for event in streaming_agent.run(prompt, stream=True, stream_events=True):
        yield event                    # 转发流式事件

    session_state["content_plans"].append({...})
    yield StepOutput(content=final_content)  # 最终输出
```

### 两个 Workflow 对比

```python
# 同步版
content_creation_workflow = Workflow(
    steps=[research_step, content_planning_step, content_summary_step],
    session_state={"content_plans": [], "plan_counter": 0},
)

# 流式版
streaming_content_workflow = Workflow(
    steps=[research_step, stream_content_planning_step, stream_content_summary_step],
    session_state={"content_plans": [], "plan_counter": 0},
)
```

### 跨 run 状态累积

```python
# 第一次运行
content_creation_workflow.print_response(input="AI trends")
# session_state: {"content_plans": [{"id": 1, ...}], "plan_counter": 1}

# 第二次运行（同一 session）
content_creation_workflow.print_response(input="ML tools")
# session_state: {"content_plans": [{"id": 1, ...}, {"id": 2, ...}], "plan_counter": 2}
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/step.py` | `Step._execute_executor()` | 检测并注入 `session_state` 参数 |
| `agno/workflow/workflow.py` | `Workflow.session_state` | 持久化到 SqliteDb |
| `agno/run/workflow.py` | `WorkflowRunOutputEvent` | 流式事件类型 |
