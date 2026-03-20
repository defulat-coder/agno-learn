# tool_use.py — 实现原理分析

> 源文件：`cookbook/90_models/nvidia/tool_use.py`

## 概述

本示例展示 **`Nvidia` + WebSearchTools**，同步、流式与异步流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nvidia(id="meta/llama-3.3-70b-instruct")` | Chat |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | 默认 |

用户消息：`"Whats happening in France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["print_response"] --> B["【关键】工具 + Nvidia"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nvidia/` | `Nvidia` |
