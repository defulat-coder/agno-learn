# tool_use.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/tool_use.py`

## 概述

**`gemini-2.0-flash-001` + WebSearchTools**，同步/异步流式演示。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-2.0-flash-001")` | |
| `tools` | `[WebSearchTools()]` | |
| `markdown` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["工具循环"] --> B["【关键】generate_content + tools"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `invoke` | |
