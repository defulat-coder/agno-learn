# knowledge.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Run `uv pip install ddgs sqlalchemy pgvector pypdf openai` to install dependencies."""

from agno.agent import Agent
from agno.knowledge.embedder.azure_openai import AzureOpenAIEmbedder
from agno.knowledge.knowledge import Knowledge
from agno.models.azure import AzureAIFoundry
from agno.vectordb.pgvector import PgVector

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

knowledge = Knowledge(
    vector_db=PgVector(
        table_name="recipes",
        db_url=db_url,
        embedder=AzureOpenAIEmbedder(),
    ),
)
# Add content to the knowledge
knowledge.insert(url="https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf")

agent = Agent(
    model=AzureAIFoundry(id="Cohere-command-r-08-2024"),
    knowledge=knowledge,
)
agent.print_response("How to make Thai curry?", markdown=True)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/azure/ai_foundry/knowledge.py`

## 概述

**Knowledge + PgVector + AzureOpenAIEmbedder + AzureAIFoundry(Cohere-command-r)**，RAG 与聊天均经 Foundry。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureAIFoundry(id="Cohere-command-r-08-2024")` | 生成 |
| `knowledge` | `Knowledge(..., embedder=AzureOpenAIEmbedder())` | 检索 |

## System Prompt 组装

默认 `search_knowledge=True` 时注入知识说明；检索正文动态。

## Mermaid 流程图

```mermaid
flowchart TD
    K["PgVector + embedder"] --> A["【关键】知识上下文 + Foundry"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/_messages.py` | `# 3.3.13` | 知识 |
| `agno/models/azure/ai_foundry.py` | `invoke()` | complete |
