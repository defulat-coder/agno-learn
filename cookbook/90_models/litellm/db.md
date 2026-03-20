# db.md — 实现原理分析

> 源文件：`cookbook/90_models/litellm/db.py`

## 概述

**`SqliteDb` + `LiteLLM(gpt-4o)` + WebSearch + 历史**，两轮对话。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LiteLLM(id="gpt-4o")` | LiteLLM |
| `db` | `SqliteDb(db_file="tmp/data.db")` | 开发用 SQLite |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `add_history_to_context` | `True` | 历史 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["SqliteDb"] --> B["【关键】history messages"]
    B --> C["LiteLLM.completion"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/db/sqlite.py` | SqliteDb |
| `agno/models/litellm/chat.py` | `invoke` |
