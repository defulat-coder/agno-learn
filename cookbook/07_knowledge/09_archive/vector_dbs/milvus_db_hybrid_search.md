# milvus_db_hybrid_search.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Milvus Hybrid Search
====================

Demonstrates Milvus hybrid search with sync and async flows.
"""

import asyncio

from agno.agent import Agent
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.milvus import Milvus, SearchType

# ---------------------------------------------------------------------------
# Setup
# ---------------------------------------------------------------------------
vector_db = Milvus(
    collection="recipes",
    uri="/tmp/milvus_hybrid.db",
    search_type=SearchType.hybrid,
)


# ---------------------------------------------------------------------------
# Create Knowledge Base
# ---------------------------------------------------------------------------
knowledge = Knowledge(vector_db=vector_db)


# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
agent = Agent(knowledge=knowledge)


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
def run_sync() -> None:
    knowledge.insert(url="https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf")
    agent.print_response("How to make Tom Kha Gai", markdown=True)


async def run_async() -> None:
    await knowledge.ainsert(
        url="https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf"
    )
    await agent.aprint_response("How to make Tom Kha Gai", markdown=True)


if __name__ == "__main__":
    run_sync()
    asyncio.run(run_async())
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/07_knowledge/09_archive/vector_dbs/milvus_db_hybrid_search.py`

## 概述

**`Milvus`** + **`SearchType.hybrid`**；同步/异步各执行一次 **`insert` 食谱 + 问 Tom Kha Gai**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `uri` | `/tmp/milvus_hybrid.db` | |

## 核心组件解析

Milvus 混合检索融合稠密与稀疏通道（实现见适配器）。

## System Prompt 组装

默认 knowledge 段。

## 完整 API 请求

默认 `gpt-4o`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["SearchType.hybrid"] --> B["【关键】Milvus hybrid search"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/vectordb/milvus/` | `SearchType` |
