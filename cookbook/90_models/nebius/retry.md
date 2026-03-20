# retry.py — 实现原理分析

> 源文件：`cookbook/90_models/nebius/retry.py`

## 概述

本示例展示 **Nebius 模型重试参数**与错误 `nebius-wrong-id`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nebius(id=wrong_model_id, retries=3, delay_between_retries=1, exponential_backoff=True)` | 重试 |

用户消息：`"What is the capital of France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["invoke 失败"] --> B["【关键】退避重试"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/nebius/nebius.py` | `Nebius` |
