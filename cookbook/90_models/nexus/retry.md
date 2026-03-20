# retry.py — 实现原理分析

> 源文件：`cookbook/90_models/nexus/retry.py`

## 概述

本示例展示 **Nexus 模型重试**与错误 id `nexus-wrong-id`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nexus(..., retries=3, delay_between_retries=1, exponential_backoff=True)` | 重试 |

用户消息：`"What is the capital of France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["失败"] --> B["【关键】重试退避"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nexus/nexus.py` | `Nexus` |
