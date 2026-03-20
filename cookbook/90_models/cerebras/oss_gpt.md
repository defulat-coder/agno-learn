# oss_gpt.py — 实现原理分析

> 源文件：`cookbook/90_models/cerebras/oss_gpt.py`

## 概述

**Cerebras** 使用 **`gpt-oss-120b`** 与 **WebSearchTools**，演示 OSS GPT 类模型在 Cerebras 上的工具调用。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cerebras(id="gpt-oss-120b")` | OSS GPT |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | Markdown |

## Mermaid 流程图

```mermaid
flowchart TD
    A["gpt-oss-120b"] --> B["【关键】工具循环"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/cerebras/cerebras.py` | `get_request_params` | tools |
