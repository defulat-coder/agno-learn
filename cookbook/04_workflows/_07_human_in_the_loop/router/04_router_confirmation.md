# 04_router_confirmation.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Router with Confirmation HITL Example

This example demonstrates the confirmation mode for Router components,
which is different from the user selection mode.

When `requires_confirmation=True` on a Router (with a selector):
- The selector determines which steps to run
- User is asked to confirm before executing those steps
- User confirms -> Execute the selected steps
- User rejects -> Skip the router entirely

This is useful for:
- Confirming automated routing decisions
- Human oversight of programmatic selections
- Safety checks before executing routed steps

Note: This is different from `requires_user_input=True` which lets the user
choose which steps to execute. Here, the selector chooses, but user confirms.
"""

from agno.db.sqlite import SqliteDb
from agno.workflow.router import Router
from agno.workflow.step import Step
from agno.workflow.types import StepInput, StepOutput
from agno.workflow.workflow import Workflow


# ============================================================
# Step functions
# ============================================================
def analyze_request(step_input: StepInput) -> StepOutput:
    """Analyze the request to determine routing."""
    user_query = step_input.input or "general request"
    # Determine category based on content
    if "urgent" in user_query.lower():
        category = "urgent"
    elif "billing" in user_query.lower():
        category = "billing"
    else:
        category = "general"

    return StepOutput(
        content=f"Request analyzed:\n"
        f"- Query: {user_query}\n"
        f"- Detected category: {category}\n"
        "- Ready for routing"
    )


def handle_urgent(step_input: StepInput) -> StepOutput:
    """Handle urgent requests."""
    return StepOutput(
        content="Urgent Request Handling:\n"
        "- Priority escalation initiated\n"
        "- Immediate response generated\n"
        "- Notification sent to on-call team"
    )


def handle_billing(step_input: StepInput) -> StepOutput:
    """Handle billing requests."""
    return StepOutput(
        content="Billing Request Handling:\n"
        "- Account details retrieved\n"
        "- Billing history analyzed\n"
        "- Response prepared"
    )


def handle_general(step_input: StepInput) -> StepOutput:
    """Handle general requests."""
    return StepOutput(
        content="General Request Handling:\n"
        "- Standard processing applied\n"
        "- Response generated"
    )


def finalize_response(step_input: StepInput) -> StepOutput:
    """Finalize the response."""
    previous_content = step_input.previous_step_content or "No handling"
    return StepOutput(
        content=f"=== FINAL RESPONSE ===\n\n{previous_content}\n\nRequest processed successfully."
    )


# Selector function that determines routing based on previous step content
def route_by_category(step_input: StepInput) -> str:
    """Route based on detected category in previous step."""
    content = step_input.previous_step_content or ""
    if "urgent" in content.lower():
        return "handle_urgent"
    elif "billing" in content.lower():
        return "handle_billing"
    else:
        return "handle_general"


# Define the steps
analyze_step = Step(name="analyze_request", executor=analyze_request)

# Router with confirmation - selector chooses, user confirms
request_router = Router(
    name="request_router",
    choices=[
        Step(
            name="handle_urgent",
            description="Handle urgent requests",
            executor=handle_urgent,
        ),
        Step(
            name="handle_billing",
            description="Handle billing requests",
            executor=handle_billing,
        ),
        Step(
            name="handle_general",
            description="Handle general requests",
            executor=handle_general,
        ),
    ],
    selector=route_by_category,
    requires_confirmation=True,
    confirmation_message="The system has selected a handler. Proceed with the routed action?",
)

finalize_step = Step(name="finalize_response", executor=finalize_response)

# Create workflow with database for HITL persistence
workflow = Workflow(
    name="router_confirmation_demo",
    steps=[analyze_step, request_router, finalize_step],
    db=SqliteDb(db_file="tmp/router_hitl.db"),
)

if __name__ == "__main__":
    print("=" * 60)
    print("Router with Confirmation HITL Example")
    print("=" * 60)
    print("The selector will choose the route, but you must confirm.")
    print()

    # Test with an urgent request
    run_output = workflow.run("URGENT: System is down!")

    # Handle HITL pauses
    while run_output.is_paused:
        # Handle Step requirements (confirmation for router)
        for requirement in run_output.steps_requiring_confirmation:
            print(f"\n[ROUTING DECISION] {requirement.step_name}")
            print(f"[HITL] {requirement.confirmation_message}")

            user_choice = input("\nProceed with routing? (yes/no): ").strip().lower()
            if user_choice in ("yes", "y"):
                requirement.confirm()
                print("[HITL] Confirmed - executing routed steps")
            else:
                requirement.reject()
                print("[HITL] Rejected - skipping router")

        run_output = workflow.continue_run(run_output)

    print("\n" + "=" * 60)
    print(f"Status: {run_output.status}")
    print("=" * 60)
    print(run_output.content)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/router/04_router_confirmation.py`

## 概述

本示例展示 Agno **Router 确认型 HITL**：由 **`selector` 函数自动决定**走哪条分支，但在执行前**暂停并要求用户确认**（`requires_confirmation=True`）；确认则执行路由结果，拒绝则跳过整个 Router。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `Workflow.name` | `"router_confirmation_demo"` | 工作流名 |
| `Workflow.db` | `SqliteDb(db_file="tmp/router_hitl.db")` | 持久化 |
| `Router.name` | `"request_router"` | 路由器 |
| `Router.choices` | `handle_urgent` / `handle_billing` / `handle_general` | 候选步骤 |
| `Router.selector` | `route_by_category` | 根据上一步内容返回 step 名字符串 |
| `Router.requires_confirmation` | `True` | 执行前确认 |
| `Router.confirmation_message` | `"The system has selected a handler. Proceed with the routed action?"` | 确认文案 |
| `Router.requires_user_input` | 未设置（默认 `False`） | 非用户选路模式 |
| `Agent` | 无 | 无 LLM |

## 架构分层

```
用户代码层                agno.workflow 层
┌──────────────────┐    ┌──────────────────────────────────┐
│ workflow.run()   │───>│ analyze_request → selector 选路   │
│ steps_requiring_ │    │  → 【暂停】confirmation           │
│   confirmation   │    │  confirm()/reject() → continue_run│
└──────────────────┘    └──────────────────────────────────┘
```

## 核心组件解析

### selector 与确认的分工

`Router` 文档（`router.py`）说明：`requires_user_input` 与 `requires_confirmation` 是不同模式。本例中 `route_by_category` 解析 `previous_step_content` 中的 `urgent`/`billing` 等关键字，返回对应 step 名；**用户不选路**，只在 `steps_requiring_confirmation` 上调用 `confirm` 或 `reject`。

### 运行机制与因果链

1. **路径**：`workflow.run("URGENT: ...")` → `analyze_request` 产出含 `urgent` 的文本 → selector 指向 `handle_urgent` → 暂停待确认 → 用户 yes → 执行 `handle_urgent` → `finalize_response`。
2. **副作用**：DB 记录暂停点；reject 时 Router 按 `on_reject` 策略跳过（默认 `skip`）。
3. **分支**：确认 vs 拒绝；与 `requires_user_input=True` 的 Router 对比。
4. **差异**：同目录 `01` 是用户选路；本例是**自动选路 + 人工确认**。

## System Prompt 组装

无 LLM Agent。`confirmation_message` 面向人类操作者，不进入 `get_system_message()`。

### 还原后的完整 System 文本

```text
（无模型 system。）
```

### 段落释义

不适用。

## 完整 API 请求

无大模型调用。

## Mermaid 流程图

```mermaid
flowchart TD
    A["analyze_request"] --> B["selector 自动选路"]
    B --> C["【关键】Router requires_confirmation"]
    C --> D{{"confirm?"}}
    D -->|yes| E["执行所选 handler Step"]
    D -->|no| F["跳过 Router"]
    E --> G["finalize_response"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/workflow/router.py` | `Router` `requires_confirmation` | 确认型 HITL |
| `agno/workflow/workflow.py` | `continue_run` | 续跑 |
