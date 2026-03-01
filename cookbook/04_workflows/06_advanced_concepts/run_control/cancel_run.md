# cancel_run.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/run_control/cancel_run.py`

## 概述

本示例展示 Agno Workflow **跨线程取消运行中的 Workflow**：在一个线程中流式运行 Workflow（长时任务），在另一个线程中延迟几秒后调用 `workflow.cancel_run(run_id)` 触发取消，Workflow 流式输出中会出现 `WorkflowRunEvent.workflow_cancelled` 事件，实现生产级的任务取消控制。

**核心 API：**

| API | 说明 |
|-----|------|
| `workflow.run(stream=True)` | 流式运行（在线程中） |
| `chunk.run_id` | 从流式 chunk 获取 run_id |
| `workflow.cancel_run(run_id)` | 取消指定 run |
| `WorkflowRunEvent.workflow_cancelled` | 取消成功的流式事件 |
| `RunEvent.run_cancelled` | 步骤级取消事件 |

## 核心组件解析

### 多线程取消模式

```python
run_id_container = {}

# 线程 1：流式运行，获取 run_id
def long_running_task(workflow, run_id_container):
    for chunk in workflow.run("Write a very long story...", stream=True):
        if "run_id" not in run_id_container and chunk.run_id:
            run_id_container["run_id"] = chunk.run_id  # 保存 run_id

        if chunk.event == WorkflowRunEvent.workflow_cancelled:
            # 处理取消事件
            return

# 线程 2：延迟后取消
def cancel_after_delay(workflow, run_id_container, delay_seconds=3):
    time.sleep(delay_seconds)
    run_id = run_id_container.get("run_id")
    if run_id:
        success = workflow.cancel_run(run_id)   # 取消运行
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.cancel_run(run_id)` | 取消指定 run |
| `agno/run/workflow.py` | `WorkflowRunEvent.workflow_cancelled` | 取消事件 |
