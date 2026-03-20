# filtering_chroma_db.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
from agno.agent import Agent
from agno.knowledge.knowledge import Knowledge
from agno.utils.media import (
    SampleDataFileExtension,
    download_knowledge_filters_sample_data,
)
from agno.vectordb.chroma import ChromaDb

# Download all sample CVs and get their paths
downloaded_cv_paths = download_knowledge_filters_sample_data(
    num_files=5, file_extension=SampleDataFileExtension.PDF
)

# Initialize ChromaDB
vector_db = ChromaDb(collection="recipes", path="tmp/chromadb", persistent_client=True)

# Step 1: Initialize knowledge with documents and metadata
# ------------------------------------------------------------------------------
# When initializing the knowledge, we can attach metadata that will be used for filtering
# This metadata can include user IDs, document types, dates, or any other attributes

knowledge = Knowledge(
    name="ChromaDB Knowledge Base",
    description="A knowledge base for ChromaDB",
    vector_db=vector_db,
)

# Load all documents into the vector database
knowledge.insert_many(
    [
        {
            "path": downloaded_cv_paths[0],
            "metadata": {
                "user_id": "jordan_mitchell",
                "document_type": "cv",
                "year": 2025,
            },
        },
        {
            "path": downloaded_cv_paths[1],
            "metadata": {
                "user_id": "taylor_brooks",
                "document_type": "cv",
                "year": 2025,
            },
        },
        {
            "path": downloaded_cv_paths[2],
            "metadata": {
                "user_id": "morgan_lee",
                "document_type": "cv",
                "year": 2025,
            },
        },
        {
            "path": downloaded_cv_paths[3],
            "metadata": {
                "user_id": "casey_jordan",
                "document_type": "cv",
                "year": 2025,
            },
        },
        {
            "path": downloaded_cv_paths[4],
            "metadata": {
                "user_id": "alex_rivera",
                "document_type": "cv",
                "year": 2025,
            },
        },
    ]
)
# Step 2: Query the knowledge base with different filter combinations
# ------------------------------------------------------------------------------

agent = Agent(
    knowledge=knowledge,
    search_knowledge=True,
)

agent.print_response(
    "Tell me about Jordan Mitchell's experience and skills",
    knowledge_filters={"user_id": "jordan_mitchell"},
    markdown=True,
)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/07_knowledge/09_archive/filters/filtering_chroma_db.py`

## 概述

**ChromaDB** 上 `insert_many` PDF CV + metadata，`Agent` + `knowledge_filters`（如按 `user_id`）。持久化路径 `tmp/chromadb`。

## System Prompt 组装

默认 Agent。

## 完整 API 请求

默认 Chat 模型。

## Mermaid 流程图

```mermaid
flowchart TD
    A["ChromaDb"] --> B["【关键】knowledge_filters 检索"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/vectordb/chroma` | Chroma |
