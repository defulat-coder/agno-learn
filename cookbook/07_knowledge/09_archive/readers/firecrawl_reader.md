# firecrawl_reader.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
import os

from agno.knowledge.reader.firecrawl_reader import FirecrawlReader

api_key = os.getenv("FIRECRAWL_API_KEY")

reader = FirecrawlReader(
    api_key=api_key,
    mode="scrape",
    chunk=True,
    # for crawling
    # params={
    #     'limit': 5,
    #     'scrapeOptions': {'formats': ['markdown']}
    # }
    # for scraping
    params={"formats": ["markdown"]},
)

try:
    print("Starting scrape...")
    documents = reader.read("https://github.com/agno-agi/agno")

    if documents:
        for doc in documents:
            print(doc.name)
            print(doc.content)
            print(f"Content length: {len(doc.content)}")
            print("-" * 80)
    else:
        print("No documents were returned")

except Exception as e:
    print(f"Error type: {type(e)}")
    print(f"Error occurred: {str(e)}")
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/07_knowledge/09_archive/readers/firecrawl_reader.py`

## 概述

仅调用 **`FirecrawlReader`** 抓取 GitHub 页面，打印文档块；**无 Knowledge / Agent**，需 **`FIRECRAWL_API_KEY`**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `FirecrawlReader` | `mode="scrape"`, `chunk=True`, `params` | scrape 模式 |
| LLM | 无 | |

## 核心组件解析

### Firecrawl 集成

将 URL 转为 `Document` 列表，供后续自行接入 `Knowledge.insert`。

## System Prompt 组装

无 Agent。

## 完整 API 请求

无 LLM；仅有 Firecrawl HTTP API（由 Reader 发起）。

## Mermaid 流程图

```mermaid
flowchart TD
    A["FIRECRAWL_API_KEY"] --> B["【关键】Firecrawl scrape"]
    B --> C[print documents]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/reader/firecrawl_reader.py` | |
