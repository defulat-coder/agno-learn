# event_storage.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/run_control/event_storage.py`

## 概述

本示例展示 Agno Workflow **事件存储与过滤机制**：`Workflow(store_events=True, events_to_skip=[...])` 在流式执行时持久化 Workflow 事件，同时排除高频噪音事件（如 `RunContent`、`StepStarted`）。完成后通过 `workflow.get_last_run_output().events` 获取已存储事件列表，可用于调试、审计或重放。

**核心 API：**

| 配置/API | 说明 |
|---------|------|
| `Workflow(store_events=True)` | 启用事件存储 |
| `events_to_skip=[...]` | 跳过指定事件类型 |
| `workflow.get_last_run_output()` | 获取最近一次运行的输出 |
| `run_output.events` | 已存储的事件列表 |

## 核心组件解析

### 事件存储配置

```python
step_workflow = Workflow(
    steps=[research_step, search_step],
    db=SqliteDb(session_table="workflow_session", db_file="tmp/workflow.db"),
    store_events=True,
    events_to_skip=[
        WorkflowRunEvent.step_started,
        WorkflowRunEvent.workflow_completed,
        RunEvent.run_content,       # 跳过高频内容事件
        RunEvent.run_started,
        RunEvent.run_completed,
    ],
)

# 获取存储的事件
run_response = step_workflow.get_last_run_output()
for event in run_response.events:
    print(event.event)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.store_events` / `events_to_skip` | 事件存储控制 |
| `agno/workflow/workflow.py` | `Workflow.get_last_run_output()` | 获取最近运行输出 |
| `agno/run/workflow.py` | `WorkflowRunOutput.events` | 存储的事件列表 |
