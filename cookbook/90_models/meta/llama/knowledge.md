# knowledge.md — 实现原理分析

> 源文件：`cookbook/90_models/meta/llama/knowledge.py`

## 概述

**`Llama` + Knowledge(PgVector)**，Thai curry。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `Llama(id="Llama-4-Maverick-17B-128E-Instruct-FP8")` | Meta |
| `knowledge` | `Knowledge(...)` | RAG |

## Mermaid 流程图

```mermaid
flowchart TD
    A["Knowledge"] --> B["【关键】Llama.chat.completions"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/meta/llama.py` | `invoke` L212+ |
