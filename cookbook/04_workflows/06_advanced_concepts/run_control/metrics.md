# metrics.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/run_control/metrics.py`

## 概述

本示例展示 Agno Workflow **从 `WorkflowRunOutput` 读取运行级和步骤级 Metrics**：通过 `workflow_run_response.metrics` 访问整体执行时长，通过 `metrics.steps` 字典按步骤名访问各步骤的 token 用量和执行时长，实现精细化的性能监控。

**核心配置一览：**

| API | 说明 |
|-----|------|
| `workflow_run_response.metrics` | Workflow 整体 metrics |
| `metrics.duration` | 总执行时长（秒） |
| `metrics.steps` | `Dict[str, StepMetrics]` 按步骤名 |
| `step_metrics.metrics.duration` | 步骤执行时长 |
| `step_metrics.metrics.total_tokens` | 步骤 token 用量 |

## 核心组件解析

### 读取 WorkflowRunOutput metrics

```python
from agno.run.workflow import WorkflowRunOutput

workflow_run_response: WorkflowRunOutput = content_creation_workflow.run(
    input="AI trends in 2024"
)

if workflow_run_response.metrics:
    # Workflow 整体 metrics
    print(json.dumps(workflow_run_response.metrics.to_dict(), indent=2))

    # 总执行时长
    if workflow_run_response.metrics.duration:
        print(f"Total: {workflow_run_response.metrics.duration:.2f} seconds")

    # 步骤级 metrics
    for step_name, step_metrics in workflow_run_response.metrics.steps.items():
        print(f"Step: {step_name}")
        if step_metrics.metrics:
            print(f"  Duration: {step_metrics.metrics.duration:.2f}s")
            print(f"  Tokens: {step_metrics.metrics.total_tokens}")
```

### vs workflow_with_session_metrics.py 对比

| 特性 | metrics.py | workflow_with_session_metrics.py |
|------|-----------|--------------------------------|
| 读取方式 | `run_response.metrics` | `workflow.get_session_metrics()` |
| 范围 | 单次 run | 整个 session（多次 run 聚合） |
| 步骤粒度 | `metrics.steps[name]` | `SessionMetrics` 按 session |
| 适用场景 | 监控单次执行 | 分析历史趋势 |

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/run/workflow.py` | `WorkflowRunOutput.metrics` | 运行级 metrics |
| `agno/run/workflow.py` | `WorkflowRunOutput.metrics.steps` | 步骤级 metrics 字典 |
