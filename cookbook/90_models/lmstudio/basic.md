# basic.md — 实现原理分析

> 源文件：`cookbook/90_models/lmstudio/basic.py`

## 概述

**`LMStudio` 默认连接 `http://127.0.0.1:1234/v1`**（`lmstudio.py`），同步与流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LMStudio(id="qwen2.5-7b-instruct-1m")` | 本地 LM Studio |
| `markdown` | `True` | Markdown |

## Mermaid 流程图

```mermaid
flowchart TD
    A["LMStudio"] --> B["【关键】:1234/v1 Chat Completions"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/lmstudio/lmstudio.py` | `LMStudio` L7+，`supports_json_schema_outputs` |
