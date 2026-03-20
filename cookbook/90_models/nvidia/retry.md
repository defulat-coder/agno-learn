# retry.py — 实现原理分析

> 源文件：`cookbook/90_models/nvidia/retry.py`

## 概述

本示例展示 **Nvidia 模型重试**与 `nvidia-wrong-id`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nvidia(id=wrong_model_id, retries=3, delay_between_retries=1, exponential_backoff=True)` | 重试 |

用户消息：`"What is the capital of France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["invoke"] --> B["【关键】重试"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nvidia/` | `Nvidia` |
