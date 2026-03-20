# agentos_excel_analyst.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Agentos Excel Analyst
=====================

Demonstrates agentos excel analyst.
"""

from pathlib import Path

from agno.agent import Agent
from agno.db.postgres import PostgresDb
from agno.knowledge.knowledge import Knowledge
from agno.knowledge.reader.excel_reader import ExcelReader
from agno.models.openai import OpenAIChat
from agno.os import AgentOS
from agno.vectordb.pgvector import PgVector

# ---------------------------------------------------------------------------
# Create Example
# ---------------------------------------------------------------------------

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

db = PostgresDb(db_url=db_url)

excel_knowledge = Knowledge(
    name="Excel Products",
    contents_db=db,  # Required for UI to show knowledge
    vector_db=PgVector(
        db_url=db_url,
        table_name="agentos_excel_knowledge",
    ),
)

excel_agent = Agent(
    name="Excel Data Agent",
    model=OpenAIChat(id="gpt-4o-mini"),
    db=db,  # For session storage
    knowledge=excel_knowledge,
    search_knowledge=True,
    markdown=True,
    instructions=[
        "You are a data analyst assistant with access to Excel spreadsheet data.",
        "Search the knowledge base to answer questions about the data.",
        "Provide specific numbers and details when available.",
    ],
)

# Create AgentOS app
agent_os = AgentOS(
    description="Excel Knowledge API - Query Excel data via REST",
    agents=[excel_agent],
)

app = agent_os.get_app()

# ---------------------------------------------------------------------------
# Run Example
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    repo_root = Path(__file__).parent.parent.parent.parent
    sample_file = (
        repo_root / "cookbook/07_knowledge/testing_resources/sample_products.xlsx"
    )

    if sample_file.exists():
        print("Loading sample products data...")
        excel_knowledge.insert(
            path=str(sample_file),
            reader=ExcelReader(),
            skip_if_exists=True,
        )

    print("\nStarting AgentOS server...")
    print("Test at: http://localhost:7777/")
    print("\nExample queries:")
    print("  - What electronics products are in stock?")
    print("  - What is the price of the Bluetooth speaker?")

    agent_os.serve(app="agentos_excel_analyst:app", reload=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/05_agent_os/knowledge/agentos_excel_analyst.py`

## 概述

本示例展示 Agno 的 **AgentOS + Postgres/PgVector + ExcelReader 入库 + 检索问答** 机制：`Knowledge` 指向 `contents_db` 与 `PgVector`，启动时用 `ExcelReader` 将 `sample_products.xlsx` 写入向量库；Agent 显式 `search_knowledge=True`，面向表格数据问答。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `db` | `PostgresDb` | 会话与内容元数据 |
| `knowledge` | `Knowledge` + `PgVector(table_name=agentos_excel_knowledge)` | 向量检索 |
| `search_knowledge` | `True` | 启用检索与 `search_knowledge_base` 工具 |
| `model` | `OpenAIChat(id="gpt-4o-mini")` | Chat Completions |
| `instructions` | 多行 | 分析师角色 |

## 架构分层

```
Excel → knowledge.insert(reader=ExcelReader) → PgVector
用户请求 → Agent → search_knowledge_base / hybrid → LLM 回答
```

## 运行机制与因果链

1. **入库**：`insert(path=..., reader=ExcelReader(), skip_if_exists=True)`。
2. **检索**：`# 3.3.13` 将 knowledge `build_context` 拼入 system；默认工具 `search_knowledge_base`（`_default_tools.py`）。

## System Prompt 组装

### 还原后的 instructions 字面量

```text
You are a data analyst assistant with access to Excel spreadsheet data.
Search the knowledge base to answer questions about the data.
Provide specific numbers and details when available.
```

## 完整 API 请求

`OpenAIChat.invoke`，含 tools（含 `search_knowledge_base`）与检索结果注入。

## Mermaid 流程图

```mermaid
flowchart TD
    A["Excel 入库"] --> B["PgVector"]
    B --> C["【关键】search_knowledge_base"]
    C --> D["gpt-4o-mini"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/knowledge/reader/excel_reader.py` | `ExcelReader` | 表格解析 |
| `agno/agent/_default_tools.py` | `search_knowledge_base` | 检索工具 |
| `agno/agent/_messages.py` | `# 3.3.13` | 知识段 |
