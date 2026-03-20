# url_context.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/url_context.py`

## 概述

**`url_context=True`**：模型拉取并理解 URL 内容，对比两则菜谱 URL。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-2.5-flash", url_context=True)` | |
| `markdown` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["url_context=True"] --> B["【关键】URL 内容进上下文"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `url_context` | |
