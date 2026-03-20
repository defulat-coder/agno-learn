# basic.md — 实现原理分析

> 源文件：`cookbook/90_models/meta/llama_openai/basic.py`

## 概述

**`LlamaOpenAI`** 全模式调用，与 `meta/llama/basic.py` 平行，走 **OpenAI 兼容** 客户端（`llama_openai.py` 继承 `OpenAILike`）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LlamaOpenAI(id="Llama-4-Maverick-17B-128E-Instruct-FP8")` | Meta OpenAI 端点 |
| `markdown` | `True` | Markdown |

## Mermaid 流程图

```mermaid
flowchart TD
    A["LlamaOpenAI"] --> B["【关键】OpenAILike chat.completions"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/meta/llama_openai.py` | `LlamaOpenAI` |
