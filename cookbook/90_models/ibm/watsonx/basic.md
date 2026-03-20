# basic.py — 实现原理分析

> 源文件：`cookbook/90_models/ibm/watsonx/basic.py`

## 概述

本示例展示 **`WatsonX`** 的同步/流式/异步调用，`markdown=True`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `WatsonX(id="mistralai/mistral-small-3-1-24b-instruct-2503")` | WatsonX |
| `markdown` | `True` | Markdown 附加段 |

## System Prompt 组装

静态段含：

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

用户消息：`Share a 2 sentence horror story`

## 完整 API 请求

见 `watsonx.py`：`client.chat(messages=formatted_messages, **request_params)`。

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/ibm/watsonx.py` | `invoke` |
