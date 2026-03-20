# references_format.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
References Format
=============================

Control how knowledge base references are formatted for the agent.

By default, references are returned as JSON. Set references_format="yaml"
to return them as YAML instead.
"""

from agno.agent import Agent
from agno.knowledge.embedder.openai import OpenAIEmbedder
from agno.knowledge.knowledge import Knowledge
from agno.models.openai import OpenAIResponses
from agno.vectordb.pgvector import PgVector, SearchType

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"
knowledge = Knowledge(
    vector_db=PgVector(
        table_name="recipes_yaml_demo",
        db_url=db_url,
        search_type=SearchType.hybrid,
        embedder=OpenAIEmbedder(id="text-embedding-3-small"),
    ),
)

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
agent = Agent(
    model=OpenAIResponses(id="gpt-5.2"),
    knowledge=knowledge,
    search_knowledge=True,
    # Format knowledge references as YAML instead of the default JSON
    references_format="yaml",
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    knowledge.insert(url="https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf")
    agent.print_response(
        "How do I make chicken and galangal in coconut milk soup?",
        stream=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/02_agents/07_knowledge/references_format.py`

## 概述

本示例展示 **`references_format`** 参数：控制知识库检索结果注入模型上下文时的 **序列化格式**，`"yaml"` 替代默认 JSON，便于阅读或下游解析。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(id="gpt-5.2")` | Responses API |
| `knowledge` | `PgVector` hybrid + `OpenAIEmbedder` | 同系列 RAG |
| `search_knowledge` | `True` | Agentic |
| `references_format` | `"yaml"` | 引用块格式 |
| `markdown` | `True` | 是 |

## 核心组件解析

`references_format` 在 `Knowledge.get_tools` / 文档转字符串路径中传入（见 `knowledge.py` 中 `references_format` 与 `convert_documents` 相关逻辑），影响 `<references>` 或工具结果中的展示。

### 运行机制与因果链

检索仍走 hybrid；**仅改变引用文档的编码格式**，不改变向量索引本身。

## System Prompt 组装

含标准 knowledge 段与 markdown；无自定义 `instructions`。

参照用户句：`How do I make chicken and galangal in coconut milk soup?`

## 完整 API 请求

`responses.create`；YAML 格式化发生在 **组装消息/工具结果** 阶段，而非 API 层。

## Mermaid 流程图

```mermaid
flowchart LR
    D["检索文档"] --> F["【关键】convert_documents<br/>references_format=yaml"]
    F --> U["注入 user/工具上下文"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/knowledge.py` | `references_format` 处理 |
| `agno/agent/_utils.py` | `convert_documents_to_string` 等 |
