# 02_custom_function_step_confirmation.py — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/confirmation/02_custom_function_step_confirmation.py`

## 概述

本示例展示 Agno Workflow **`@pause` 装饰器方式的 HITL 确认**：在自定义函数步骤上使用 `@pause(requires_confirmation=True)` 装饰器，Workflow 执行到该步骤时自动暂停，等待用户确认后再继续。对比 `Step(requires_confirmation=True)` 的标志位方式，装饰器方式更适合函数型自定义步骤。

**两种 HITL 确认方式对比：**

| 方式 | 适用场景 | 配置 |
|------|---------|------|
| `Step(requires_confirmation=True)` | Agent/Team 步骤 | 在 Step 上设置 |
| `@pause(requires_confirmation=True)` | 自定义函数步骤 | 在函数上装饰 |

## 核心组件解析

### @pause 装饰器

```python
from agno.workflow.decorators import pause

@pause(
    name="Process Research",
    requires_confirmation=True,
    confirmation_message="Research complete. Ready to generate blog post. Proceed?",
)
def process_research(step_input: StepInput) -> StepOutput:
    research = step_input.previous_step_content or "No research available"
    return StepOutput(content=f"PROCESSED RESEARCH:\n{research}\n\nReady for blog post generation.")

# Step 自动检测 @pause 装饰器
process_step = Step(name="process_research", executor=process_research)
```

### HITL 暂停处理

```python
run_output = workflow.run("Benefits of morning exercise")

while run_output.is_paused:
    for requirement in run_output.steps_requiring_confirmation:
        user_input = input("Continue? (yes/no): ").strip().lower()
        if user_input in ("yes", "y"):
            requirement.confirm()
        else:
            requirement.reject()

    run_output = workflow.continue_run(run_output)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/decorators.py` | `@pause` | 函数步骤 HITL 装饰器 |
| `agno/run/workflow.py` | `WorkflowRunOutput.is_paused` | 检查暂停状态 |
| `agno/run/workflow.py` | `WorkflowRunOutput.steps_requiring_confirmation` | 待确认的步骤列表 |
