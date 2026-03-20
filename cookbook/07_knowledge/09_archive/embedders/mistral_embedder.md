# mistral_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/embedders/mistral_embedder.py`

## 概述

**`MistralEmbedder`** + `PgVector` 表 `mistral_embeddings`，可选 `enable_batch`；`get_embedding` 与 `ainsert`。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

Mistral Embeddings API。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】MistralEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/mistral.py` | Mistral |
