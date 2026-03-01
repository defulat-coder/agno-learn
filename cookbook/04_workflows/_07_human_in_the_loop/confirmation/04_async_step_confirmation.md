# 04_async_step_confirmation.md — 实现原理分析

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/confirmation/04_async_step_confirmation.py`

## 概述

本示例展示 Agno Workflow **异步模式下 `@pause` 装饰器与 `async` 函数的兼容性**：`@pause` 通过直接附加元数据到函数对象（而非包装函数），保留了 `async` 函数的异步特性，配合 `workflow.arun()` 和 `workflow.acontinue_run()` 实现完整的异步 HITL 流程。

**关键点：**

| 特性 | 说明 |
|------|------|
| `@pause` + `async def` | 装饰器不包装函数，异步特性完整保留 |
| `workflow.arun()` | 异步执行 |
| `workflow.acontinue_run()` | 异步继续执行 |
| `AsyncPostgresDb` | 异步模式需要异步 DB |

## 核心组件解析

```python
from agno.db.postgres import AsyncPostgresDb

@pause(
    name="Async Data Processor",
    requires_confirmation=True,
    confirmation_message="Ready to process asynchronously. Continue?",
)
async def async_process_data(step_input: StepInput) -> StepOutput:
    await asyncio.sleep(0.5)  # 模拟异步 I/O
    return StepOutput(content="ASYNC PROCESSED:\n...")

workflow = Workflow(
    db=AsyncPostgresDb(db_url=async_db_url),  # 异步 DB
    steps=[research_step, Step(name="async_process", executor=async_process_data), write_step],
)

async def main():
    run_output = await workflow.arun("Benefits of meditation")

    while run_output.is_paused:
        for req in run_output.steps_requiring_confirmation:
            req.confirm()
        run_output = await workflow.acontinue_run(run_output)  # 异步继续
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/decorators.py` | `@pause` | 元数据附加方式不破坏 async |
| `agno/workflow/workflow.py` | `Workflow.acontinue_run()` | 异步继续执行 |
| `agno/db/postgres.py` | `AsyncPostgresDb` | 异步 PostgreSQL 支持 |
