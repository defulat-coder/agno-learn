# db.py — 实现原理分析

> 源文件：`cookbook/90_models/openai/chat/db.py`

## 概述

**Postgres + gpt-4o + WebSearchTools + add_history_to_context** 多轮。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | Chat |
| `db` | `PostgresDb` | 持久化 |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `add_history_to_context` | `True` | 历史 |

用户消息：加拿大人口与国歌。

## Mermaid 流程图

```mermaid
flowchart TD
    A["轮次1"] --> B["【关键】轮次2 历史消息"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/_messages.py` | `get_run_messages` |
