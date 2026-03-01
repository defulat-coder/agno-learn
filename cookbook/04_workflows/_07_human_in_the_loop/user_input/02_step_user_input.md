# 02_step_user_input.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/user_input/02_step_user_input.py`

## 概述

本示例展示 Agno Workflow **在 Step 级别直接配置用户输入 HITL**（无需 `@pause` 装饰器）：`Step(requires_user_input=True, user_input_schema=[UserInputField(...)])` 在 Agent 步骤或函数步骤上均可使用，暂停时提示用户填写结构化字段，用户填写后通过 `requirement.set_user_input(**values)` 提交，输入数据自动追加到 Agent 消息或通过 `step_input.additional_data["user_input"]` 传递给函数。

**两种 HITL 用户输入方式对比：**

| 方式 | 配置位置 | 适用场景 |
|------|---------|---------|
| `@pause(requires_user_input=True)` | 函数装饰器 | 函数型步骤 |
| `Step(requires_user_input=True)` | Step 级别 | Agent 或函数步骤 |

## 核心组件解析

### Step 级别用户输入配置

```python
from agno.workflow.types import UserInputField

Step(
    name="generate_content",
    agent=content_agent,
    requires_user_input=True,
    user_input_message="Please provide your content preferences:",
    user_input_schema=[
        UserInputField(name="tone", field_type="str",
                       description="'formal', 'casual', or 'technical'", required=True),
        UserInputField(name="length", field_type="str",
                       description="'short', 'medium', or 'long'", required=True),
        UserInputField(name="include_examples", field_type="bool", required=False),
    ],
)
```

### 收集并提交用户输入

```python
for requirement in run_output.steps_requiring_user_input:
    user_values = {}
    for field in requirement.user_input_schema:
        value = input(f"{field.name}: ").strip()
        if field.field_type == "bool":
            user_values[field.name] = value.lower() in ("true", "yes", "1", "y")
        else:
            user_values[field.name] = value

    requirement.set_user_input(**user_values)

run_output = workflow.continue_run(run_output)
```

### 函数步骤获取用户输入

```python
def process_data(step_input: StepInput) -> StepOutput:
    # 用户输入通过 additional_data["user_input"] 传入
    user_input = step_input.additional_data.get("user_input", {})
    format_type = user_input.get("format", "json")
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/types.py` | `UserInputField` | 用户输入字段定义 |
| `agno/run/workflow.py` | `WorkflowRunOutput.steps_requiring_user_input` | 待输入的步骤列表 |
| 输入 requirement | `.set_user_input(**values)` | 提交用户输入 |
