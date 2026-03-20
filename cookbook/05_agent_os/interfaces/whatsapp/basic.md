# basic.py — 实现原理分析

> 源文件：`cookbook/05_agent_os/interfaces/whatsapp/basic.py`

## 概述

本示例展示 Agno 的 **WhatsApp + OpenAI Chat 最小助手** 机制：`OpenAIChat(gpt-4o)`、`SqliteDb` 持久会话、`add_history_to_context` 与 `markdown`，无工具。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | Chat Completions |
| `db` | `SqliteDb` | 会话 |
| `add_history_to_context` | `True`，`num_history_runs=3` | 是 |
| `add_datetime_to_context` | `True` | 是 |
| `markdown` | `True` | 是 |

## 架构分层

```
WhatsApp → Whatsapp 适配器 → Agent → OpenAIChat.invoke
```

## System Prompt 组装

无 `instructions`/`description`；含时间、markdown 提示等默认段。

## 完整 API 请求

`client.chat.completions.create(model="gpt-4o", messages=[...])`

## Mermaid 流程图

```mermaid
flowchart TD
    A["WhatsApp 文本"] --> B["【关键】Agent + 历史"]
    B --> C["OpenAIChat.invoke"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/openai/chat.py` | `invoke()` | Chat Completions |
| `agno/os/interfaces/whatsapp` | `Whatsapp` | 通道 |
