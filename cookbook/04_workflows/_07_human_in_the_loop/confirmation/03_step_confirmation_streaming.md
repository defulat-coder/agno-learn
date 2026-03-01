# 03_step_confirmation_streaming.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/confirmation/03_step_confirmation_streaming.py`

## 概述

本示例展示 Agno Workflow **流式模式下的 Step HITL 确认**：使用 `workflow.run(stream=True, stream_events=True)` 流式执行，当步骤到达确认点时发出 `StepPausedEvent` 事件，用户确认后通过 `workflow.continue_run(run_output, stream=True, stream_events=True)` 继续流式执行。

**流式 HITL 与非流式的关键差异：**

| 对比点 | 非流式 | 流式 |
|--------|--------|------|
| 初始运行 | `workflow.run()` | `workflow.run(stream=True, stream_events=True)` |
| 暂停通知 | `run_output.is_paused` | `StepPausedEvent` 事件 |
| 获取 run_output | 直接返回 | `workflow.get_session().runs[-1]` |
| 继续执行 | `workflow.continue_run(run_output)` | `workflow.continue_run(run_output, stream=True, stream_events=True)` |

## 核心组件解析

### 流式 HITL 完整流程

```python
from agno.run.workflow import StepPausedEvent, WorkflowCompletedEvent

workflow = Workflow(
    steps=[
        Step(name="fetch_data", agent=fetch_agent),
        Step(
            name="process_data",
            agent=process_agent,
            requires_confirmation=True,
            on_reject=OnReject.skip,   # 拒绝时跳过而非取消
        ),
        Step(name="save_results", agent=save_agent),
    ],
)

# 流式运行
event_stream = workflow.run("Process user data", stream=True, stream_events=True)
for event in event_stream:
    if isinstance(event, StepPausedEvent):
        print(f"Step paused: {event.step_name}")   # 检测暂停事件

# 从 session 获取 run_output
session = workflow.get_session()
run_output = session.runs[-1]

# 处理确认并继续
while run_output and run_output.is_paused:
    for req in run_output.steps_requiring_confirmation:
        req.confirm()   # 或 req.reject()
    continue_stream = workflow.continue_run(run_output, stream=True, stream_events=True)
    for event in continue_stream:
        ...   # 继续处理事件
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/run/workflow.py` | `StepPausedEvent` | 步骤暂停流式事件 |
| `agno/workflow/workflow.py` | `Workflow.get_session()` | 从 DB 获取当前会话 |
| `agno/workflow/workflow.py` | `Workflow.continue_run(stream=True)` | 流式继续执行 |
| `agno/workflow/__init__.py` | `OnReject` | 拒绝行为枚举 |
