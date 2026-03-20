# db.md — 实现原理分析

> 源文件：`cookbook/90_models/meta/llama/db.py`

## 概述

**`Llama` + PostgresDb + WebSearch + 历史**，两轮对话。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `Llama(id="Llama-4-Maverick-17B-128E-Instruct-FP8")` | Meta |
| `db` | `PostgresDb(...)` | 会话 |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `add_history_to_context` | `True` | 历史 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["PostgresDb"] --> B["【关键】history + Llama"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/meta/llama.py` | `Llama` |
