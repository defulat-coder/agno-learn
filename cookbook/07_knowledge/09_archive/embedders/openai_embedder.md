# openai_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/embedders/openai_embedder.py`

## 概述

**`OpenAIEmbedder()`** + `PgVector` 表 `openai_embeddings`，可选 `enable_batch`；`get_embedding` 与 `ainsert`。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

OpenAI `embeddings.create`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】OpenAIEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/openai.py` | OpenAI |
