# knowledge.py — 实现原理分析

> 源文件：`cookbook/90_models/cerebras/knowledge.py`

## 概述

**Knowledge + PgVector（默认 embedder）+ Cerebras**，RAG 默认 `search_knowledge=True`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cerebras(id="llama-3.3-70b")` | 生成 |
| `knowledge` | `Knowledge(vector_db=PgVector(...))` | 检索 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["knowledge"] --> B["【关键】build_context + Cerebras"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/_messages.py` | `# 3.3.13` | 知识 |
