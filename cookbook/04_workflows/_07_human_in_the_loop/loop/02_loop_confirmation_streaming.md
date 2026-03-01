# 02_loop_confirmation_streaming.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/loop/02_loop_confirmation_streaming.py`

## 概述

本示例展示 Agno Workflow **流式模式下 Loop 的 HITL 确认**：`Loop(requires_confirmation=True)` 在首次迭代前暂停等待用户确认，流式执行时通过 `StepPausedEvent` 事件通知暂停，通过 `workflow.get_session().runs[-1]` 获取 `run_output`，然后调用 `workflow.continue_run(stream=True, stream_events=True)` 流式继续。

**核心区别（流式 vs 非流式）：**

| 关键点 | 非流式 | 流式 |
|--------|--------|------|
| run_output 获取 | `workflow.run()` 直接返回 | `workflow.get_session().runs[-1]` |
| 继续执行 | `workflow.continue_run(run_output)` | `workflow.continue_run(run_output, stream=True, stream_events=True)` |
| 暂停通知 | `run_output.is_paused` | `StepPausedEvent` 事件 |

## 核心组件解析

```python
refinement_loop = Loop(
    name="refinement_loop",
    steps=[Step(name="refine_analysis", executor=refine_analysis)],
    max_iterations=5,
    requires_confirmation=True,
    confirmation_message="Start the refinement loop? This may take several iterations.",
)

# 流式运行
event_stream = workflow.run("Process quarterly data", stream=True, stream_events=True)
process_event_stream(event_stream)

# 从 session 获取 run_output（流式模式必须如此）
session = workflow.get_session()
run_output = session.runs[-1]

while run_output and run_output.is_paused:
    for req in run_output.steps_requiring_confirmation:
        req.confirm()  # 或 req.reject()
    continue_stream = workflow.continue_run(run_output, stream=True, stream_events=True)
    process_event_stream(continue_stream)
    session = workflow.get_session()
    run_output = session.runs[-1]
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/loop.py` | `Loop.requires_confirmation` | Loop HITL 确认 |
| `agno/run/workflow.py` | `StepPausedEvent` | 流式暂停事件 |
| `agno/workflow/workflow.py` | `Workflow.get_session()` | 获取会话（含 run_output） |
