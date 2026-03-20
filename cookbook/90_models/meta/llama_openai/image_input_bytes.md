# image_input_bytes.md — 实现原理分析

> 源文件：`cookbook/90_models/meta/llama_openai/image_input_bytes.py`

## 概述

与 `meta/llama/image_input_bytes.py` **模型类对调**：此处为 **`Llama` + WebSearch**（非 `LlamaOpenAI`），图像 bytes + 新闻请求。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `Llama(id="Llama-4-Maverick-17B-128E-Instruct-FP8")` | Meta API |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | Markdown |

## Mermaid 流程图

```mermaid
flowchart TD
    A["Llama + 图像 bytes"] --> B["【关键】视觉 + 搜索"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/meta/llama.py` | `Llama` |
