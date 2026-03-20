# spider_tools.py — 实现原理分析

> 源文件：`cookbook/91_tools/spider_tools.py`

## 概述

本示例展示 **`SpiderTools`** 与 **`optional_params`**（如 `proxy_enabled`），以及限制为搜索-only 的 `enable_crawl=False` 配置。

**核心配置一览（`agent_all`）**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Spider Agent - All Functions"` |  |
| `tools` | `[SpiderTools(optional_params={"proxy_enabled": True})]` |  |
| `instructions` | `["You have access to all Spider web scraping capabilities."]` |  |
| `markdown` | `True` |  |

## System Prompt 组装

```text
- You have access to all Spider web scraping capabilities.

<additional_information>
- Use markdown to format your answers.
</additional_information>
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["optional_params"] --> B["Spider 网络请求"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/spider/` | `SpiderTools` |
