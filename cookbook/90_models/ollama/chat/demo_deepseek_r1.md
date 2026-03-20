# demo_deepseek_r1.py — 实现原理分析

> 源文件：`cookbook/90_models/ollama/chat/demo_deepseek_r1.py`

## 概述

**`Ollama(id="deepseek-r1:14b")`** 演示推理型本地模型写代码并解释。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Ollama(id="deepseek-r1:14b")` | 原生 chat |
| `markdown` | `True` | 默认 |

用户消息：`"Write me python code to solve quadratic equations. Explain your reasoning."`

## Mermaid 流程图

```mermaid
flowchart TD
    A["deepseek-r1"] --> B["【关键】可显式推理链（视模型）"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/ollama/chat.py` | `Ollama` |
