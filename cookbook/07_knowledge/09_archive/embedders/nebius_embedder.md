# nebius_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/embedders/nebius_embedder.py`

## 概述

**`NebiusEmbedder()`** + `PgVector` 表 `nebius_embeddings`，`get_embedding` 后 `ainsert`。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

Nebius 兼容 OpenAI 的 Embeddings 端点。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】NebiusEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/nebius.py` | Nebius |
