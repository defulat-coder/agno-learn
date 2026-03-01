# 03_router_nested_choices.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/router/03_router_nested_choices.py`

## 概述

本示例展示 Agno Workflow **Router 嵌套选择（步骤套餐）**：`Router.choices` 支持 `Step`、`Steps` 容器或嵌套列表（自动转换为 `Steps`）作为选项，用户选择某个选项时，若该选项为 `Steps` 容器，则顺序执行其中所有步骤，实现"套餐式"工作流（Basic/Standard/Premium）。

**Router choices 类型支持：**

| 选项类型 | 执行行为 |
|---------|---------|
| `Step` | 执行单个步骤 |
| `Steps(steps=[a, b])` | 顺序执行 a、b |
| `[a, b]`（嵌套列表） | 自动转为 `Steps`，顺序执行 |

## 核心组件解析

### 套餐式 Router 配置

```python
from agno.workflow.steps import Steps

quick_scan_step = Step(name="quick_scan", executor=quick_scan)

standard_package = Steps(
    name="standard_package",
    description="Deep Analysis + Quality Check (6 min)",
    steps=[
        Step(name="deep_analysis", executor=deep_analysis),
        Step(name="quality_check", executor=quality_check),
    ],
)

Router(
    name="package_selector",
    choices=[
        quick_scan_step,     # 单步套餐
        standard_package,    # 2 步套餐（Steps 容器）
        premium_package,     # 4 步套餐（Steps 容器）
    ],
    requires_user_input=True,
    user_input_message="Select a processing package:",
    allow_multiple_selections=False,   # 只选一个套餐
)

# 或使用嵌套列表（自动命名为 steps_group_N）
Router(choices=[
    Step(name="quick_scan", executor=quick_scan),
    [Step(name="deep_analysis", executor=deep_analysis),
     Step(name="quality_check", executor=quality_check)],  # -> steps_group_1
])
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/router.py` | `Router.choices` | 支持 Step/Steps/嵌套列表 |
| `agno/workflow/steps.py` | `Steps` | 步骤容器（顺序执行） |
