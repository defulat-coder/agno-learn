# tool_use.py — 实现原理分析

> 源文件：`cookbook/90_models/deepseek/tool_use.py`

## 概述

**`deepseek-chat` + WebSearchTools**；注释说明当前 FC 能力不稳定可能死循环或空回复。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `DeepSeek(id="deepseek-chat")` | |
| `tools` | `[WebSearchTools()]` | |
| `markdown` | `True` | |

## 运行机制与因果链

与 `thinking_tool_calls.py`（reasoner）对照：本文件用 chat 型号做基础工具演示。

## Mermaid 流程图

```mermaid
flowchart TD
    A["deepseek-chat"] --> B["【关键】工具循环（可能不稳定）"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/tools/websearch.py` | `WebSearchTools` | |
