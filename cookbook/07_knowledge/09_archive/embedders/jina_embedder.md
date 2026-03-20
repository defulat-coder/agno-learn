# jina_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/embedders/jina_embedder.py`

## 概述

**`JinaEmbedder(late_chunking=True, timeout=30.0)`** + `PgVector` 表 `jina_embeddings`；可选 batch 注释。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

Jina AI Embeddings API。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】JinaEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/jina.py` | Jina |
