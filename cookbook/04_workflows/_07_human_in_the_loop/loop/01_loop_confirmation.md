# 01_loop_confirmation.py — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/loop/01_loop_confirmation.py`

## 概述

本示例展示 Agno Workflow **`Loop` 的 HITL 启动确认**机制：通过 `Loop(requires_confirmation=True)` 使循环在第一次迭代前暂停，等待用户确认是否启动，用户确认则执行循环（至 `max_iterations` 或 `end_condition`），用户拒绝则跳过整个循环继续后续步骤。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `Loop.requires_confirmation` | `True` | 启动前需要用户确认 |
| `Loop.confirmation_message` | 提示文本 | 显示给用户的确认消息 |
| 拒绝行为 | 跳过整个 Loop | Loop 不执行，继续后续步骤 |

## 核心组件解析

### Loop HITL 配置

```python
refinement_loop = Loop(
    name="refinement_loop",
    steps=[Step(name="refine_analysis", executor=refine_analysis)],
    max_iterations=5,
    requires_confirmation=True,  # 启动前需要确认
    confirmation_message="Start the refinement loop? This may take several iterations.",
)
```

### HITL 交互处理

```python
run_output = workflow.run("Process quarterly data")

while run_output.is_paused:
    for requirement in run_output.steps_requiring_confirmation:
        user_choice = input("Start the loop? (yes/no): ")
        if user_choice.lower() in ("yes", "y"):
            requirement.confirm()   # 确认 → 执行循环
        else:
            requirement.reject()    # 拒绝 → 跳过循环

    run_output = workflow.continue_run(run_output)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/loop.py` | `Loop.requires_confirmation` | 启用 Loop 启动确认 |
| `agno/workflow/types.py` | `StepRequirement.confirm()/reject()` | 用户确认/拒绝接口 |
| `agno/workflow/workflow.py` | `Workflow.continue_run()` | 用户响应后继续 |
