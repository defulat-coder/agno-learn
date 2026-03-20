# basic.py — 实现原理分析

> 源文件：`cookbook/90_models/cerebras_openai/basic.py`

## 概述

本示例展示 **`CerebrasOpenAI`**（继承 **`OpenAILike`**，`base_url` 指向 Cerebras OpenAI 兼容端点）与 **`llama-4-scout-17b-16e-instruct`**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `CerebrasOpenAI(id="llama-4-scout-17b-16e-instruct")` | OpenAI 兼容 |
| `markdown` | `True` | Markdown |

## 完整 API 请求

```python
# OpenAILike: chat.completions.create — CerebrasOpenAI 覆写 get_request_params 以处理 tools 与 parallel_tool_calls（cerebras_openai.py L59–75）
```

## System Prompt 组装

### 还原后的完整 System 文本

```text
Use markdown to format your answers.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["Agent"] --> B["【关键】Cerebras OpenAI 兼容 API"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/cerebras/cerebras_openai.py` | `CerebrasOpenAI` | 认证与 tools |
