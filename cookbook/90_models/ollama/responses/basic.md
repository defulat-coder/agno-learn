# basic.py — 实现原理分析

> 源文件：`cookbook/90_models/ollama/responses/basic.py`

## 概述

**`OllamaResponses`**：走 OpenAI **Responses API** 兼容层（`/v1/responses`），非原生 `chat()`。状态无链式 `previous_response_id`（见 `ollama/responses.py` 文档字符串）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OllamaResponses(id="gpt-oss:20b")` | `OpenResponses` 子类 |
| `markdown` | `True` | 默认 |

用户消息：horror story / poem 等。

## 完整 API 请求

使用 Responses 端点（`responses.create` 形态），非 `chat.completions.create`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["OllamaResponses"] --> B["【关键】/v1/responses"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/ollama/responses.py` | `OllamaResponses` |
| `agno/models/openai/open_responses.py` | `OpenResponses` |
