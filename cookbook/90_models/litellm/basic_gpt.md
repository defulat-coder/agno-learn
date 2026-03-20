# basic_gpt.md — 实现原理分析

> 源文件：`cookbook/90_models/litellm/basic_gpt.py`

## 概述

**`LiteLLM(id="gpt-4o", name="LiteLLM")`** 最小调用。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LiteLLM(id="gpt-4o", name="LiteLLM")` | OpenAI 路由 |
| `markdown` | `True` | Markdown |

## System Prompt 组装

Markdown 附加段。用户消息：`Share a 2 sentence horror story`

## 完整 API 请求

`completion` → 通常 OpenAI 兼容 `gpt-4o`。

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/litellm/chat.py` | `invoke` |
