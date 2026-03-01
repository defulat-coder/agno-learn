# 02_error_retry_skip_streaming.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/error/02_error_retry_skip_streaming.py`

## 概述

本示例展示 Agno Workflow **流式模式下步骤错误 HITL（重试或跳过）**：`Step(on_error=OnError.pause)` 在步骤抛出异常时暂停 Workflow 并等待用户决策；用户可调用 `error_req.retry()` 重试或 `error_req.skip()` 跳过并继续，流式模式下通过 `workflow.get_session().runs[-1]` 获取 `run_output`，然后 `workflow.continue_run(stream=True, stream_events=True)` 继续执行。

**错误 HITL 关键 API：**

| API | 说明 |
|-----|------|
| `Step(on_error=OnError.pause)` | 步骤失败时暂停 |
| `run_output.steps_with_errors` | 失败步骤列表 |
| `error_req.error_message` | 错误信息 |
| `error_req.retry_count` | 已重试次数 |
| `error_req.retry()` | 重试该步骤 |
| `error_req.skip()` | 跳过该步骤继续执行 |

## 核心组件解析

```python
from agno.workflow import OnError

workflow = Workflow(
    steps=[
        Step(
            name="fetch_data",
            executor=unreliable_api_call,  # 99% 概率失败
            on_error=OnError.pause,         # 失败时暂停
        ),
        Step(name="process_data", executor=process_data),
        Step(name="save_results", executor=save_results),
    ],
)

# 流式运行
event_stream = workflow.run("Fetch and process data", stream=True, stream_events=True)
for event in event_stream:
    ...  # 处理事件

# 从 session 获取 run_output（流式必须如此）
session = workflow.get_session()
run_output = session.runs[-1]

# 处理错误 HITL
while run_output and run_output.is_paused:
    for error_req in run_output.steps_with_errors:
        print(f"Step '{error_req.step_name}' FAILED: {error_req.error_message}")
        user_choice = input("retry/skip: ").strip().lower()
        if user_choice == "retry":
            error_req.retry()
        else:
            error_req.skip()

    continue_stream = workflow.continue_run(run_output, stream=True, stream_events=True)
    for event in continue_stream:
        ...
    session = workflow.get_session()
    run_output = session.runs[-1]
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/__init__.py` | `OnError` | 错误处理策略枚举 |
| `agno/run/workflow.py` | `WorkflowRunOutput.steps_with_errors` | 失败步骤列表 |
| 错误 requirement | `.retry()` / `.skip()` | 重试或跳过失败步骤 |
