# 01_steps_pipeline_confirmation.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/steps/01_steps_pipeline_confirmation.py`

## 概述

本示例展示 Agno Workflow **`Steps` 容器的 HITL 确认**机制：通过 `Steps(requires_confirmation=True)` 使整个步骤管道作为一个整体等待用户确认，用户确认则顺序执行容器内的所有步骤（验证→转换→丰富），用户拒绝则跳过整个管道，适用于可选的高成本处理流程。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `Steps.requires_confirmation` | `True` | 整个步骤管道需要确认 |
| `Steps.confirmation_message` | 提示文本 | 显示给用户的确认消息 |
| 拒绝行为 | 跳过整个 Steps 管道 | 内部步骤均不执行 |

## 核心组件解析

### Steps 管道 HITL 配置

```python
from agno.workflow.steps import Steps

advanced_processing = Steps(
    name="advanced_processing_pipeline",
    steps=[
        Step(name="validate_data", executor=validate_data),
        Step(name="transform_data", executor=transform_data),
        Step(name="enrich_data", executor=enrich_data),
    ],
    requires_confirmation=True,                 # 整体确认
    confirmation_message="Run advanced processing pipeline? (validation + transformation + enrichment)",
)
```

### 组合到 Workflow

```python
workflow = Workflow(
    steps=[
        collect_step,          # 自动执行
        advanced_processing,   # HITL 确认后执行（3 个步骤）
        report_step,           # 自动执行
    ],
)
```

### HITL 模式对比

| 目标 | 配置方式 |
|------|---------|
| 确认单个步骤 | `Step(requires_confirmation=True)` |
| 确认整个 Loop | `Loop(requires_confirmation=True)` |
| 确认整个管道 | `Steps(requires_confirmation=True)` |
| 用户选路由 | `Router(requires_user_input=True)` |

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/steps.py` | `Steps.requires_confirmation` | 启用步骤管道整体确认 |
| `agno/workflow/workflow.py` | `Workflow.continue_run()` | 用户响应后继续 |
