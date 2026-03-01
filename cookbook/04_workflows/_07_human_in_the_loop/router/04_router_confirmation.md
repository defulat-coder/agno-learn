# 04_router_confirmation.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/router/04_router_confirmation.py`

## 概述

本示例展示 Agno Workflow **Router 自动路由 + 用户确认模式**：`Router(selector=route_by_category, requires_confirmation=True)` 让程序自动通过 `selector` 决定路由目标，然后暂停等待用户确认是否执行，与 `requires_user_input=True`（用户手动选择路由）形成对比。

**两种 Router HITL 模式对比：**

| 模式 | 谁选路由 | 用户操作 |
|------|---------|---------|
| `requires_user_input=True` | 用户 | 选择执行哪个步骤 |
| `requires_confirmation=True` | selector 自动选 | 确认或拒绝执行 |

## 核心组件解析

```python
def route_by_category(step_input: StepInput) -> str:
    content = step_input.previous_step_content or ""
    if "urgent" in content.lower():
        return "handle_urgent"
    elif "billing" in content.lower():
        return "handle_billing"
    return "handle_general"

request_router = Router(
    name="request_router",
    choices=[
        Step(name="handle_urgent", executor=handle_urgent),
        Step(name="handle_billing", executor=handle_billing),
        Step(name="handle_general", executor=handle_general),
    ],
    selector=route_by_category,       # 自动路由
    requires_confirmation=True,        # 执行前用户确认
    confirmation_message="The system has selected a handler. Proceed with the routed action?",
)

# 确认路由决定
for requirement in run_output.steps_requiring_confirmation:
    requirement.confirm()  # 或 requirement.reject()（跳过整个 Router）
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/router.py` | `Router.requires_confirmation` | 路由后用户确认 |
| `agno/run/workflow.py` | `WorkflowRunOutput.steps_requiring_confirmation` | 待确认的路由 |
