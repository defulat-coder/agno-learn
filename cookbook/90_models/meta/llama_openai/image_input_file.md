# image_input_file.md — 实现原理分析

> 源文件：`cookbook/90_models/meta/llama_openai/image_input_file.py`

## 概述

**`LlamaOpenAI` + `Image(filepath)`**，无工具。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LlamaOpenAI(id="Llama-4-Maverick-17B-128E-Instruct-FP8")` | OpenAI 兼容 |
| `markdown` | `True` | Markdown |

## Mermaid 流程图

```mermaid
flowchart TD
    A["LlamaOpenAI"] --> B["【关键】filepath 图像"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/meta/llama_openai.py` | `LlamaOpenAI` |
