# web_reader.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
from agno.knowledge.reader.website_reader import WebsiteReader

reader = WebsiteReader(max_depth=3, max_links=10)


try:
    print("Starting read...")
    documents = reader.read("https://docs.agno.com/introduction")
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

> 源文件：`cookbook/07_knowledge/09_archive/readers/web_reader.py`

## 概述

**`WebsiteReader(max_depth=3, max_links=10)`** 爬取 `docs.agno.com/introduction`，打印文档；**无 Agent**。

## 核心组件解析

站内跟随链接深度与数量受限，防止爬爆。

## System Prompt 组装

无 LLM。

## 完整 API 请求

仅 HTTP 抓取。

## Mermaid 流程图

```mermaid
flowchart TD
    A["WebsiteReader"] --> B["【关键】深度+链接上限"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/reader/website_reader.py` | |
