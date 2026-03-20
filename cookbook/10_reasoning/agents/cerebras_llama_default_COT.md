# cerebras_llama_default_COT.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/agents/cerebras_llama_default_COT.py`

## 概述

本示例展示 **`Cerebras(id="llama-3.3-70b")` + `reasoning=True` + `debug_mode=True`** 的默认 COT 回退行为，任务为 Fibonacci 步骤。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cerebras` | 非 OpenAI |
| `reasoning` | `True` | 内置推理 |

## 完整 API 请求

以 `agno/models/cerebras` 中客户端为准（非 `chat.completions` 若厂商不同）。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/cerebras` | Cerebras 适配 |
