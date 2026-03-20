# knowledge.py — 实现原理分析

> 源文件：`cookbook/90_models/openai/chat/knowledge.py`

## 概述

**`Knowledge` + `PgVector` + `OpenAIChat(gpt-4o)`**，默认 `search_knowledge=True`，PDF 入库后问泰式咖喱。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | Chat |
| `knowledge` | `Knowledge(vector_db=PgVector(...))` | RAG |

用户消息：`"How to make Thai curry?"`，`markdown=True` 在调用处。

## Mermaid 流程图

```mermaid
flowchart TD
    A["PgVector"] --> B["【关键】#3.3.13 知识指令"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/knowledge.py` | `Knowledge` |
