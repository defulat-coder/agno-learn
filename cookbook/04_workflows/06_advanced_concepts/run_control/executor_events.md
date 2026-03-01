# executor_events.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/run_control/executor_events.py`

## 概述

本示例展示 Agno Workflow **过滤内部执行器事件**：通过 `Workflow(stream_executor_events=False)` 在流式输出中屏蔽来自 Agent/Team 内部的 `RunContent`、`TeamRunContent` 等高频事件，只保留 Workflow 和 Step 层面的粗粒度事件，适用于只需监控 Workflow 进度而不关心内容细节的场景。

**核心配置：**

| 配置 | 值 | 说明 |
|------|------|------|
| `stream_executor_events` | `False` | 屏蔽 Agent/Team 内部事件 |
| `stream_events` | `True` | 启用事件流 |

## 核心组件解析

```python
workflow = Workflow(
    name="Research Workflow",
    steps=[Step(name="Research", agent=agent)],
    stream=True,
    stream_executor_events=False,  # 过滤 RunContent 等内部事件
)

for event in workflow.run("What is Python?", stream=True, stream_events=True):
    event_name = event.event if hasattr(event, "event") else type(event).__name__
    print(f"  -> {event_name}")
    # 只会看到 WorkflowStarted, StepStarted, StepCompleted, WorkflowCompleted 等
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.stream_executor_events` | 控制内部事件是否流式输出 |
