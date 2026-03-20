# pdf_reader_async.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
import asyncio

from agno.agent import Agent
from agno.knowledge.knowledge import Knowledge
from agno.vectordb.pgvector import PgVector

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

# Create a knowledge base with the PDFs from the data/pdfs directory
knowledge = Knowledge(
    vector_db=PgVector(
        table_name="pdf_documents",
        db_url=db_url,
    )
)

# Create an agent with the knowledge base
agent = Agent(
    knowledge=knowledge,
    search_knowledge=True,
)

if __name__ == "__main__":
    asyncio.run(
        knowledge.ainsert(
            path="cookbook/07_knowledge/testing_resources/cv_1.pdf",
        )
    )
    # Create and use the agent
    asyncio.run(
        agent.aprint_response(
            "What skills does an applicant require to apply for the Software Engineer position?",
            markdown=True,
        )
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/07_knowledge/09_archive/readers/pdf_reader_async.py`

## 概述

默认 PDF 管线：`ainsert(path=cv_1.pdf)`，无显式 **`PDFReader`**；`aprint_response` 问岗位要求。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `table_name` | `pdf_documents` | |

## 核心组件解析

依赖内置 PDF 读取与分块；密码 PDF 需另见 `pdf_reader_password.py`。

## System Prompt 组装

默认 knowledge 块。

## 完整 API 请求

异步 Chat Completions。

## Mermaid 流程图

```mermaid
flowchart TD
    A["ainsert PDF"] --> B["【关键】默认 PDF 分块嵌入"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/reader/pdf_reader.py` | 可选显式使用 |
