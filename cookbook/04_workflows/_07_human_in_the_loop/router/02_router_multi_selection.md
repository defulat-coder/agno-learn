# 02_router_multi_selection.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/router/02_router_multi_selection.py`

## 概述

本示例展示 Agno Workflow **Router HITL 多选模式**：`Router(requires_user_input=True, allow_multiple_selections=True)` 允许用户从 `choices` 中选择多个步骤，Workflow 依次（串行）执行所有已选步骤，适用于"自定义处理管道"场景（如用户自选清洗、验证、转换等步骤的组合）。

**核心配置：**

| 参数 | 值 | 说明 |
|------|------|------|
| `requires_user_input` | `True` | 暂停等待用户选择 |
| `allow_multiple_selections` | `True` | 允许多选 |
| `user_input_message` | 提示文本 | 显示给用户的选择提示 |

## 核心组件解析

```python
processing_router = Router(
    name="processing_pipeline",
    choices=[
        Step(name="clean", description="清洗数据", executor=clean_data),
        Step(name="validate", description="验证完整性", executor=validate_data),
        Step(name="enrich", description="数据丰富化", executor=enrich_data),
        Step(name="transform", description="转换", executor=transform_data),
        Step(name="aggregate", description="聚合", executor=aggregate_data),
    ],
    requires_user_input=True,
    user_input_message="Select processing steps to apply (comma-separated for multiple):",
    allow_multiple_selections=True,   # 关键：允许多选
)

# 处理用户选择
for requirement in run_output.steps_requiring_route:
    selections = ["clean", "validate", "transform"]
    if len(selections) > 1:
        requirement.select_multiple(selections)   # 多选 API
    else:
        requirement.select(selections[0])         # 单选 API
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/router.py` | `Router.allow_multiple_selections` | 多选支持 |
| `agno/run/workflow.py` | `WorkflowRunOutput.steps_requiring_route` | 待路由选择列表 |
| 路由 requirement | `.select_multiple(selections)` | 提交多选结果 |
