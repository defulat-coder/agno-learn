# retriever.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
from typing import Optional

from agno.agent import Agent
from agno.knowledge.embedder.openai import OpenAIEmbedder
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.qdrant import Qdrant
from qdrant_client import QdrantClient

# ---------------------------------------------------------
# This section loads the knowledge base. Skip if your knowledge base was populated elsewhere.
# Define the embedder
embedder = OpenAIEmbedder(id="text-embedding-3-small")
# Initialize vector database connection
vector_db = Qdrant(
    collection="thai-recipes", url="http://localhost:6333", embedder=embedder
)
# Load the knowledge base
knowledge = Knowledge(
    vector_db=vector_db,
)

knowledge.insert(
    url="https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf",
)

# ---------------------------------------------------------


# Define the custom knowledge retriever
# This is the function that the agent will use to retrieve documents
def knowledge_retriever(
    query: str, agent: Optional[Agent] = None, num_documents: int = 5, **kwargs
) -> Optional[list[dict]]:
    """
    Custom knowledge retriever function to search the vector database for relevant documents.

    Args:
        query (str): The search query string
        agent (Agent): The agent instance making the query
        num_documents (int): Number of documents to retrieve (default: 5)
        **kwargs: Additional keyword arguments

    Returns:
        Optional[list[dict]]: List of retrieved documents or None if search fails
    """
    try:
        qdrant_client = QdrantClient(url="http://localhost:6333")
        query_embedding = embedder.get_embedding(query)
        results = qdrant_client.query_points(
            collection_name="thai-recipes",
            query=query_embedding,
            limit=num_documents,
        )
        results_dict = results.model_dump()
        if "points" in results_dict:
            return results_dict["points"]
        else:
            return None
    except Exception as e:
        print(f"Error during vector database search: {str(e)}")
        return None


def main():
    """Main function to demonstrate agent usage."""
    # Initialize agent with custom knowledge retriever
    # The knowledge object is required to register the search_knowledge_base tool
    # The knowledge_retriever overrides the default retrieval logic
    agent = Agent(
        knowledge=knowledge,
        knowledge_retriever=knowledge_retriever,
        search_knowledge=True,
        instructions="Search the knowledge base for information",
    )

    # Example query
    query = "List down the ingredients to make Massaman Gai"
    agent.print_response(query, markdown=True)


if __name__ == "__main__":
    main()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/07_knowledge/09_archive/custom_retriever/retriever.py`

## 概述

**同步** `knowledge_retriever`：`QdrantClient.query_points` + `OpenAIEmbedder`，`instructions="Search the knowledge base for information"`，**无显式 model**；`print_response` 查询 Massaman Gai 配料。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `knowledge_retriever` | 同步函数 | 阻塞 Qdrant |
| `knowledge` | `Qdrant` + 先 `insert` PDF | 数据准备 |

## 架构分层

```
sync retriever → Qdrant → Agent
```

## 核心组件解析

与 `async_retriever.py` 对照：同步客户端与 `print_response` 主流程。

## System Prompt 组装

```text
Search the knowledge base for information
```
（嵌入默认 system 拼装，见 `01_custom_retriever` 同系列。）

## 完整 API 请求

默认 Model。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】sync knowledge_retriever"] --> B["print_response"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/_messages.py` | retriever 注入 |
