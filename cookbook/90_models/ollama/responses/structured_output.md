# structured_output.py — 实现原理分析

> 源文件：`cookbook/90_models/ollama/responses/structured_output.py`

## 概述

**`OllamaResponses` + `output_schema=MovieScript`**，Responses API 上的 JSON schema 格式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OllamaResponses(id="gpt-oss:20b")` | Responses |
| `description` | `"You write movie scripts."` | 字面量 |
| `output_schema` | `MovieScript` | Pydantic |

### description 原样

```text
You write movie scripts.
```

用户消息：`"New York"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["MovieScript"] --> B["【关键】Responses text.format"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/openai/responses.py` | `format` json_schema |
