# tool_use.py — 实现原理分析

> 源文件：`cookbook/90_models/openai/responses/tool_use.py`

## 概述

本示例展示 Agno 的 **`OpenAIResponses` + `WebSearchTools`** 机制：演示同步、同步流式、异步流式三种调用法国新闻问题。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(id="gpt-4o")` | Responses |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | Markdown |

## Mermaid 流程图

```mermaid
flowchart TD
    A["Whats happening in France?"] --> B["【关键】工具循环"]
    B --> C["responses.create / stream"]
```

## System Prompt 组装

### 还原后的完整 System 文本

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>

```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/openai/responses.py` | `invoke` / `ainvoke` | 异步路径 L741 |
