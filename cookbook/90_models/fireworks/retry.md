# retry.py — 实现原理分析

> 源文件：`cookbook/90_models/fireworks/retry.py`

## 概述

**Fireworks 重试**：`fireworks-wrong-id`，`retries=3` 等。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Fireworks(id="fireworks-wrong-id", retries=3, delay_between_retries=1, exponential_backoff=True)` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["invoke"] --> B["【关键】重试"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/fireworks/fireworks.py` | `Fireworks` | |
