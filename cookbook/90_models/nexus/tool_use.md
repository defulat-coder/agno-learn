# tool_use.py — 实现原理分析

> 源文件：`cookbook/90_models/nexus/tool_use.py`

## 概述

本示例展示 **Nexus + WebSearchTools**，同步/异步与流式查询法国动态。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nexus(id="anthropic/claude-sonnet-4-20250514")` | Chat |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | 默认 |

用户消息：`"Whats happening in France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["工具"] --> B["【关键】Nexus 补全"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nexus/nexus.py` | `Nexus` |
