# db.py — 实现原理分析

> 源文件：`cookbook/90_models/cerebras_openai/db.py`

## 概述

**PostgresDb + CerebrasOpenAI + WebSearchTools + 历史**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `CerebrasOpenAI(id="llama-4-scout-17b-16e-instruct")` | OpenAI 兼容 |
| `db` | `PostgresDb(...)` | 持久化 |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `add_history_to_context` | `True` | 历史 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["db"] --> B["【关键】completions + parallel_tool_calls 规则"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/cerebras/cerebras_openai.py` | `get_request_params` L72–75 | scout 强制 `parallel_tool_calls=False` |
