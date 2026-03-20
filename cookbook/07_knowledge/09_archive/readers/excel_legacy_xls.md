# excel_legacy_xls.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
from pathlib import Path

from agno.agent import Agent
from agno.knowledge.knowledge import Knowledge
from agno.knowledge.reader.excel_reader import ExcelReader
from agno.models.openai import OpenAIChat
from agno.vectordb.pgvector import PgVector

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"

# ExcelReader automatically uses xlrd for .xls files
# Date values are converted from Excel serial numbers to ISO format
# Boolean values are converted from 0/1 to True/False
reader = ExcelReader()

knowledge_base = Knowledge(
    vector_db=PgVector(
        table_name="excel_legacy_xls",
        db_url=db_url,
    ),
)

data_path = Path(__file__).parent.parent / "testing_resources" / "legacy_data.xls"

knowledge_base.insert(
    path=str(data_path),
    reader=reader,
)

agent = Agent(
    model=OpenAIChat(id="gpt-4o-mini"),
    knowledge=knowledge_base,
    search_knowledge=True,
    instructions=[
        "You are a data assistant for legacy Excel files.",
        "The workbook has two sheets: Sales Data and Inventory.",
        "Sales Data contains: Date, Product, Quantity, Price, Total.",
        "Inventory contains: Item, Available (True/False).",
        "Dates are in ISO format (YYYY-MM-DD).",
    ],
)

if __name__ == "__main__":
    print("=" * 60)
    print("Excel Legacy XLS - .xls Format Compatibility")
    print("=" * 60)

    print("\n--- Query 1: Sales records ---\n")
    agent.print_response(
        "What products were sold? Include the dates and quantities.",
        markdown=True,
        stream=True,
    )

    print("\n--- Query 2: Inventory status ---\n")
    agent.print_response(
        "Which items are currently available in inventory?",
        markdown=True,
        stream=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/07_knowledge/09_archive/readers/excel_legacy_xls.py`

## 概述

**`ExcelReader`** 通过 **xlrd** 读 **`.xls`** 老格式；示例用测试资源 **`legacy_data.xls`** 入库，并用 **多段 `instructions`** 描述表结构（Sales / Inventory）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `OpenAIChat(id="gpt-4o-mini")` | |
| `instructions` | 列表：legacy Excel、两 sheet 列说明 | |
| `insert` | `path` + `reader=ExcelReader()` | |

## 核心组件解析

### .xls 与日期/布尔

Reader 将 Excel 序列日期转 ISO，0/1 转布尔（见文件头注释）。

## System Prompt 组装

`instructions` 多段进入 `<instructions>`（默认 tag 模式下）；外加 `<knowledge_base>`。

### 可还原 instructions 摘要

见源码 L34-L40（逐字引用请以 `.py` 为准）。

## 完整 API 请求

`gpt-4o-mini` Chat Completions；`stream=True`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["ExcelReader + .xls"] --> B["【关键】xlrd 解析与表结构 instructions"]
    B --> C[print_response stream]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/reader/excel_reader.py` | xls/xlsx |
