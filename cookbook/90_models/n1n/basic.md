# basic.py — 实现原理分析

> 源文件：`cookbook/90_models/n1n/basic.py`

## 概述

本示例展示 **`N1N` 提供商 + `gpt-4o`** 的基础对话（OpenAI 兼容端点）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `N1N(id="gpt-4o")` | Chat Completions |
| `markdown` | `True` | 默认 |

## 完整 API 请求

与 OpenAI Chat 相同形态，base_url/api_key 由 `N1N` 模型类配置。

用户消息：`"Share a 2 sentence horror story."`

## Mermaid 流程图

```mermaid
flowchart TD
    A["Agent"] --> B["【关键】N1N OpenAILike"]
    B --> C["completions.create"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/n1n/` | `N1N` 定义 |
