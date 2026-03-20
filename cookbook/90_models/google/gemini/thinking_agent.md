# thinking_agent.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/thinking_agent.py`

## 概述

**`gemini-2.5-pro` + `thinking_budget=1280` + `include_thoughts=True`**，传教士过河题，同步与流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-2.5-pro", thinking_budget=1280, include_thoughts=True)` | |
| `markdown` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["thinking_budget"] --> B["【关键】思考摘要"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | thinking 参数 | |
