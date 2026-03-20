# image_agent.md — 实现原理分析

> 源文件：`cookbook/90_models/lmstudio/image_agent.py`

## 概述

**`LMStudio(id="llama3.2-vision")` + httpx 拉取图片字节 + `Image(content)`**，流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LMStudio(id="llama3.2-vision")` | 视觉模型名（本地加载） |
| `markdown` | `True` | Markdown |

用户消息：`Tell me about this image`

## Mermaid 流程图

```mermaid
flowchart TD
    A["图像 bytes"] --> B["【关键】LM Studio vision"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/lmstudio/lmstudio.py` | `LMStudio` |
