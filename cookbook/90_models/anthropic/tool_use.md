# tool_use.py — 实现原理分析

> 源文件：`cookbook/90_models/anthropic/tool_use.py`

## 概述

本示例展示 **WebSearchTools** 与 Claude 的 **同步 / 流式 / 异步** 三种 `print_response` 调用方式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Claude(id="claude-sonnet-4-20250514")` | 工具调用 |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | Markdown |

## System Prompt 组装

### 还原后的完整 System 文本

```text
Use markdown to format your answers.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["用户问题"] --> B["【关键】agentic 工具循环"]
    B --> C["Claude + WebSearch"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `print_response` / `aprint_response` | 同步异步 |
| `agno/models/anthropic/claude.py` | `invoke` | API |
