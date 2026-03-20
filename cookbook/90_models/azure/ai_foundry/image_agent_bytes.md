# image_agent_bytes.py — 实现原理分析

> 源文件：`cookbook/90_models/azure/ai_foundry/image_agent_bytes.py`

## 概述

**Image(content=bytes)** 与 **Llama-3.2-11B-Vision-Instruct**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureAIFoundry(id="Llama-3.2-11B-Vision-Instruct")` | Vision |
| `markdown` | `True` | Markdown |
| `images` | `Image(content=image_bytes)` | 字节 |

## System Prompt 组装

### 还原后的完整 System 文本

```text
Use markdown to format your answers.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["Image bytes"] --> B["【关键】Foundry vision complete"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/ai_foundry.py` | `invoke()` | complete |
