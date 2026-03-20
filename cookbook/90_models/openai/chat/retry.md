# retry.py — 实现原理分析

> 源文件：`cookbook/90_models/openai/chat/retry.py`

## 概述

**OpenAIChat + `gpt-wrong-id` + 重试三件套**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id=wrong_model_id, retries=3, delay_between_retries=1, exponential_backoff=True)` | 重试 |

用户消息：`"What is the capital of France?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["失败"] --> B["【关键】指数退避重试"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/openai/chat.py` | `OpenAIChat` |
