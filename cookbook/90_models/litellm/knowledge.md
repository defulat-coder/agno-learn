# knowledge.md — 实现原理分析

> 源文件：`cookbook/90_models/litellm/knowledge.py`

## 概述

**`LiteLLM(gpt-4o)` + Knowledge(PgVector)**，Thai curry RAG。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LiteLLM(id="gpt-4o")` | LiteLLM |
| `knowledge` | `Knowledge(PgVector(...))` | RAG |

## Mermaid 流程图

```mermaid
flowchart TD
    A["Knowledge.build_context"] --> B["【关键】LiteLLM.completion"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/agent/_messages.py` | 3.3.13 |
