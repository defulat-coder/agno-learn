# image_agent_local_file.py — 实现原理分析

> 源文件：`cookbook/90_models/cohere/image_agent_local_file.py`

## 概述

**Image(filepath=...)** 与 **c4ai-aya-vision-8b**，流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cohere(id="c4ai-aya-vision-8b")` | Vision |
| `markdown` | `True` | Markdown |
| `images` | `Image(filepath=image_path)` | 本地路径 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["filepath"] --> B["【关键】读文件进 chat"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/cohere/chat.py` | `invoke()` | 同上 |
