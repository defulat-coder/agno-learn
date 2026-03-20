# together_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/embedders/together_embedder.py`

## 概述

**`TogetherEmbedder()`** + `PgVector` 表 `together_embeddings`，`ainsert` CV。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

Together AI Embeddings API。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】TogetherEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/together.py` | Together |
