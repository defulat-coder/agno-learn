# voyageai_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/embedders/voyageai_embedder.py`

## 概述

**`VoyageAIEmbedder()`** + `PgVector` 表 `voyageai_embeddings`，可选 batch 注释；`get_embedding` 与 `ainsert`。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

Voyage AI Embeddings API。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】VoyageAIEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/voyageai.py` | Voyage |
