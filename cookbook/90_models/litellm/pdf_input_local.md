# pdf_input_local.md — 实现原理分析

> 源文件：`cookbook/90_models/litellm/pdf_input_local.py`

## 概述

**`File(filepath=...)` 本地路径 + `add_history_to_context=True`**，问具体菜谱与健康信息。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LiteLLM(id="gpt-4o")` | LiteLLM |
| `markdown` | `True` | Markdown |
| `add_history_to_context` | `True` | 历史 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["File filepath"] --> B["【关键】文件 + 历史"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/litellm/chat.py` | `invoke` |
