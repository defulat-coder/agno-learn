# 07_async_task_mode.py — 实现原理分析

> 源文件：`cookbook/03_teams/02_modes/tasks/07_async_task_mode.py`

## 概述

本示例展示 Agno 的 **tasks 模式异步执行**：使用 `asyncio.run()` 驱动 `team.arun()`，Leader 并行安排 Market Researcher 和 Trend Analyzer，结合 `asyncio.gather` 实现高并发场景。这是 tasks 模式在 I/O 密集型应用（如 API 服务）中的推荐用法。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Market Research Team"` | Team 名称 |
| `model` | `OpenAIResponses(id="gpt-5.2")` | Leader |
| `mode` | `TeamMode.tasks` | 自主任务模式 |
| `members` | `[market_researcher, trend_analyzer]` | 两名研究员（gpt-5-mini） |
| `max_iterations` | `10` | 循环上限 |
| `show_members_responses` | `True` | 显示成员响应 |

## 核心组件解析

### 异步执行入口

```python
async def main():
    response = await team.arun("Analyze smartphone market trends and predict future developments")
    print(response.content)

asyncio.run(main())
```

`arun()` 是 `run()` 的异步变体，内部通过 `asyncio.gather` 在 tasks 模式下并发执行多个成员任务，不阻塞事件循环。

### 同步 vs 异步对比

| | 同步（`print_response`） | 异步（`arun`） |
|--|--------------------------|----------------|
| 适用场景 | 脚本/CLI | API 服务/并发 |
| 阻塞 | 阻塞线程 | 非阻塞 |
| 并发成员 | asyncio 内部仍并发 | asyncio 内部并发 |
| 返回值 | 流式打印 | `RunResponse` |

## Mermaid 流程图

```mermaid
flowchart TD
    A["asyncio.run(main())"] --> B["await team.arun(...)"]
    B --> C["tasks Leader gpt-5.2"]
    C --> D["execute_tasks_parallel([...])"]

    subgraph 并发（asyncio）
        D --> E["Market Researcher<br/>gpt-5-mini"]
        D --> F["Trend Analyzer<br/>gpt-5-mini"]
    end

    E --> G["mark_all_complete(市场趋势报告)"]
    F --> G

    style A fill:#e1f5fe
    style G fill:#e8f5e9
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/team/team.py` | `arun()` | 异步执行入口 |
| `agno/team/_default_tools.py` | `execute_tasks_parallel()` | 并行任务（异步兼容） |
