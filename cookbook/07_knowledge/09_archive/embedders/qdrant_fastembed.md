# qdrant_fastembed.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
FastEmbed Embedder
==================

Demonstrates FastEmbed embeddings and knowledge insertion.
"""

import asyncio

from agno.knowledge.embedder.fastembed import FastEmbedEmbedder
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.pgvector import PgVector

# ---------------------------------------------------------------------------
# Create Knowledge Base
# ---------------------------------------------------------------------------
knowledge = Knowledge(
    vector_db=PgVector(
        db_url="postgresql+psycopg://ai:ai@localhost:5532/ai",
        table_name="qdrant_embeddings",
        embedder=FastEmbedEmbedder(),
    ),
    max_results=2,
)


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
async def main() -> None:
    embeddings = FastEmbedEmbedder().get_embedding(
        "The quick brown fox jumps over the lazy dog."
    )
    print(f"Embeddings: {embeddings[:5]}")
    print(f"Dimensions: {len(embeddings)}")

    await knowledge.ainsert(path="cookbook/07_knowledge/testing_resources/cv_1.pdf")


if __name__ == "__main__":
    asyncio.run(main())
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/07_knowledge/09_archive/embedders/qdrant_fastembed.py`

## 概述

**`FastEmbedEmbedder()`**（Qdrant FastEmbed）+ `PgVector` 表 `qdrant_embeddings`，本地轻量嵌入；`ainsert` CV。**无 Agent**。

## System Prompt 组装

无 Agent。

## 完整 API 请求

FastEmbed 本地推理，无云端 LLM。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】FastEmbedEmbedder"] --> B["ainsert"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/fastembed.py` | FastEmbed |
