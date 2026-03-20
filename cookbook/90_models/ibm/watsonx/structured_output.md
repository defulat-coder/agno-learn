# structured_output.md — 实现原理分析

> 源文件：`cookbook/90_models/ibm/watsonx/structured_output.py`

## 概述

**`WatsonX` + `output_schema=MovieScript`**，**未** 设 `use_json_mode`（与 Groq 版对比）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `WatsonX(...)` | WatsonX |
| `description` | `You help people write movie scripts.` | 角色 |
| `output_schema` | `MovieScript` | 结构化 |

## System Prompt 组装

### 还原 description 原样

```text
You help people write movie scripts.
```

框架按 `_messages.py` 3.3.15–3.3.16 追加 JSON/解析器提示（取决于 `parser_model` 与模型能力）。

用户消息：`New York`

## 完整 API 请求

`WatsonX.invoke` + `response_format` 由 Agent 传入（视结构化实现而定）。

## Mermaid 流程图

```mermaid
flowchart TD
    A["output_schema"] --> B["【关键】WatsonX + 解析"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/ibm/watsonx.py` | `_parse_provider_response` |
