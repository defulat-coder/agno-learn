# audio_input_agent.md — 实现原理分析

> 源文件：`cookbook/90_models/litellm/audio_input_agent.py`

## 概述

**`LiteLLM(id="gpt-4o-audio-preview")` + `Audio` 媒体**，流式问音频内容。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LiteLLM(id="gpt-4o-audio-preview")` | 需支持音频输入 |
| `markdown` | `True` | Markdown |

## 核心组件解析

`LiteLLM.invoke`（`agno/models/litellm/chat.py` 约 L227–247）调用 `get_client().completion(**completion_kwargs)`（LiteLLM 统一入口）。

## System Prompt 组装

Markdown 附加段。用户消息：`What's the audio about?`，并传入 `audio=[Audio(...)]`。

## 完整 API 请求

```python
# litellm/chat.py：completion_kwargs["messages"] = _format_messages(...)
# get_client().completion(**completion_kwargs)
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["Audio 字节"] --> B["【关键】LiteLLM.completion + 多模态 messages"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/litellm/chat.py` | `invoke` L227+ |
