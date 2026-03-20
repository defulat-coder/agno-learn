# openai_embedder.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
OpenAI Embedder
===============

Demonstrates OpenAI embeddings and knowledge insertion, including a batching variant.
"""

import asyncio

from agno.knowledge.embedder.openai import OpenAIEmbedder
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.pgvector import PgVector


# ---------------------------------------------------------------------------
# Create Knowledge Base
# ---------------------------------------------------------------------------
def create_knowledge() -> Knowledge:
    # Standard mode
    embedder = OpenAIEmbedder()

    # Batching mode (uncomment to use)
    # embedder = OpenAIEmbedder(enable_batch=True, batch_size=100)

    return Knowledge(
        vector_db=PgVector(
            db_url="postgresql+psycopg://ai:ai@localhost:5532/ai",
            table_name="openai_embeddings",
            embedder=embedder,
        ),
        max_results=2,
    )


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
async def main() -> None:
    embeddings = OpenAIEmbedder().get_embedding(
        "The quick brown fox jumps over the lazy dog."
    )
    print(f"Embeddings: {embeddings[:5]}")
    print(f"Dimensions: {len(embeddings)}")

    knowledge = create_knowledge()
    await knowledge.ainsert(path="cookbook/07_knowledge/testing_resources/cv_1.pdf")


if __name__ == "__main__":
    asyncio.run(main())
```

<!-- cookbook-py-source:end -->

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
