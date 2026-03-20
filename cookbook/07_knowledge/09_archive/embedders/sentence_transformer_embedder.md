# sentence_transformer_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/embedders/sentence_transformer_embedder.py`

## 概述

**`SentenceTransformerEmbedder()`** + `PgVector` 表 `sentence_transformer_embeddings`，`ainsert` CV。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

sentence-transformers 本地推理。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】SentenceTransformerEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/sentence_transformer.py` | ST |
