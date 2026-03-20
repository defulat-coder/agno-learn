# db.py — 实现原理分析

> 源文件：`cookbook/90_models/groq/db.py`

## 概述

**PostgresDb + WebSearch + 多轮**：加拿大人口与国歌，`add_history_to_context=True`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Groq(id="llama-3.3-70b-versatile")` | |
| `db` | `PostgresDb(db_url=...)` | |
| `tools` | `[WebSearchTools()]` | |
| `add_history_to_context` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["会话持久化"] --> B["【关键】第二问依赖历史"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/db/postgres/postgres.py` | `PostgresDb` | |
| `agno/models/groq/groq.py` | `invoke()` | |
