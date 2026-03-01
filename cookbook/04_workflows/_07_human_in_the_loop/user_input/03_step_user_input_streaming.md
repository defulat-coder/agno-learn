# 03_step_user_input_streaming.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/user_input/03_step_user_input_streaming.py`

## 概述

本示例展示 Agno Workflow **流式模式下 Step 级别用户输入 HITL**，与 `02_step_user_input.py` 相比，核心差异在于：
1. 使用 `workflow.run(stream=True, stream_events=True)` 流式执行
2. 通过 `StepPausedEvent` 检测 HITL 暂停（同时展示暂停原因：用户输入/确认）
3. `UserInputField` 支持 `allowed_values` 字段值验证
4. `requirement.set_user_input()` 内置验证，抛出 `ValueError` 时需重新提示

**新增特性：`UserInputField.allowed_values` 字段验证：**

```python
UserInputField(
    name="tone",
    field_type="str",
    allowed_values=["formal", "casual", "technical"],  # 限制合法值
    required=True,
)
```

## 核心组件解析

```python
# 流式执行，检测 StepPausedEvent
event_stream = workflow.run(input_text, stream=True, stream_events=True)
for event in event_stream:
    if isinstance(event, StepPausedEvent):
        print(f"Step PAUSED: {event.step_name}")
        if event.requires_user_input:
            print(f"Message: {event.user_input_message}")

# 从 session 获取 run_output
session = workflow.get_session()
run_output = session.runs[-1]

# 处理用户输入（含验证）
while run_output and run_output.is_paused:
    for requirement in run_output.steps_requiring_user_input:
        try:
            requirement.set_user_input(tone="formal", length="short")
        except ValueError as e:
            print(f"Validation error: {e}")  # allowed_values 校验失败

    continue_stream = workflow.continue_run(run_output, stream=True, stream_events=True)
    for event in continue_stream:
        ...
    session = workflow.get_session()
    run_output = session.runs[-1]
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/types.py` | `UserInputField.allowed_values` | 字段值合法性验证 |
| `agno/run/workflow.py` | `StepPausedEvent.requires_user_input` | 暂停原因区分 |
| `agno/workflow/workflow.py` | `Workflow.get_session()` | 流式后获取 run_output |
