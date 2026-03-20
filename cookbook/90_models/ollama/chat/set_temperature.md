# set_temperature.py — 实现原理分析

> 源文件：`cookbook/90_models/ollama/chat/set_temperature.py`

## 概述

**`options={"temperature": 0.5}`** 传入 Ollama 原生请求（见 `Ollama` 对 `options` 的处理）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Ollama(id="llama3.2", options={"temperature": 0.5})` | 采样参数 |
| `markdown` | `True` | 默认 |

用户消息：`"Share a 2 sentence horror story"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["options.temperature"] --> B["【关键】_prepare_request_kwargs"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/ollama/chat.py` | `options` 合并 |
