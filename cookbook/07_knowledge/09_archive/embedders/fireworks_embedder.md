# fireworks_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/embedders/fireworks_embedder.py`

## 概述

**`FireworksEmbedder`** + `PgVector` 表 `fireworks_embeddings`，`get_embedding` 后 `ainsert`。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

Fireworks API。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】FireworksEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/fireworks.py` | Fireworks |
