# basic.md — 实现原理分析

> 源文件：`cookbook/90_models/litellm_openai/basic.py`

## 概述

**`LiteLLMOpenAI(id="gpt-4o")`** 同步与流式 horror story。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LiteLLMOpenAI(id="gpt-4o")` | OpenAI 兼容代理/路由 |
| `markdown` | `True` | Markdown |

## 完整 API 请求

`OpenAILike` → `chat.completions.create` 形态（见 `agno/models/openai/like.py`）。

## Mermaid 流程图

```mermaid
flowchart TD
    A["LiteLLMOpenAI"] --> B["【关键】OpenAILike 请求"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/litellm/litellm_openai.py` | `LiteLLMOpenAI` |
| `agno/models/openai/like.py` | `OpenAILike` |
