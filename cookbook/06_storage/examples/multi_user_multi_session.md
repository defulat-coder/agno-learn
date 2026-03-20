# multi_user_multi_session.py — 实现原理分析

> 源文件：`cookbook/06_storage/examples/multi_user_multi_session.py`

## 概述

本示例展示 **同一 Agent 实例上切换 `user_id` / `session_id`**：`SqliteDb` 持久化；`update_memory_on_run=True` 与 `num_history_runs=3` 控制记忆与历史窗口；演示多用户会话隔离与回到 user1 摘要。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(gpt-5.2)` | Chat Completions |
| `db` | `SqliteDb(tmp/data.db)` | 本地 |
| `update_memory_on_run` | `True` | 运行更新记忆 |
| `add_history_to_context` | `True` | 历史进上下文 |
| `num_history_runs` | `3` | 历史条数 |

## 架构分层

`print_response(..., user_id=..., session_id=...)` 将会话路由到不同存储键；`get_run_messages` 合并对应历史（`_messages.py` 约 L1618+）。

## 运行机制与因果链

1. **路径**：user1 两轮 → user2 两轮 → 再 user1 摘要请求。  
2. **副作用**：Sqlite 多行 session；记忆表若启用则更新。  
3. **定位**：**多租户会话键** 用法。

## System Prompt 组装

无 `instructions`；默认 system 较短。摘要请求依赖 **历史注入** 与记忆（若存在）。

## 完整 API 请求

`OpenAIChat` → `chat.completions.create`（`chat.py` L412+）。

## Mermaid 流程图

```mermaid
flowchart TD
    A["user_id + session_id"] --> B["【关键】Db 会话隔离"]
    B --> C["invoke"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/agent.py` | `print_response` 参数 |
| `agno/db/sqlite.py` | `SqliteDb` |
