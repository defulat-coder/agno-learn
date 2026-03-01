# background_poll.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/background_execution/background_poll.py`

## 概述

本示例展示 Agno Workflow **后台异步执行 + 轮询模式**：`workflow.arun(background=True)` 立即返回包含 `run_id` 的初始响应（无需等待完成），后续通过 `workflow.get_run(run_id)` 轮询执行状态，直到 `result.has_completed()` 为 True，实现非阻塞的长时任务执行。

**核心 API：**

| API | 说明 |
|-----|------|
| `workflow.arun(background=True)` | 后台启动，立即返回 |
| `bg_response.run_id` | 唯一运行 ID |
| `workflow.get_run(run_id)` | 按 ID 获取当前状态 |
| `result.has_completed()` | 检查是否执行完成 |

## 核心组件解析

### 后台执行 + 轮询

```python
async def main() -> None:
    # 后台启动（立即返回）
    bg_response = await content_creation_workflow.arun(
        input="AI trends in 2024",
        background=True,
    )
    print(f"Run ID: {bg_response.run_id}")  # 保存 run_id

    # 轮询直到完成
    while True:
        result = content_creation_workflow.get_run(bg_response.run_id)

        if result is None:
            await asyncio.sleep(5)
            continue

        if result.has_completed():
            break

        await asyncio.sleep(5)  # 每 5 秒轮询一次

    # 获取最终结果
    final_result = content_creation_workflow.get_run(bg_response.run_id)
    pprint_run_response(final_result, markdown=True)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.arun(background=True)` | 后台异步执行 |
| `agno/workflow/workflow.py` | `Workflow.get_run(run_id)` | 按 ID 获取执行状态 |
| `agno/run/workflow.py` | `WorkflowRunOutput.has_completed()` | 检查完成状态 |
