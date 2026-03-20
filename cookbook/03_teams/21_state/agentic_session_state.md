# agentic_session_state.py — 实现原理分析

> 源文件：`cookbook/03_teams/21_state/agentic_session_state.py`

## 概述

本示例展示 **`enable_agentic_state=True` + `add_session_state_to_context=True`**：Team 与成员共享 `session_state`（如 `shopping_list`），模型可通过框架提供的 **agentic 状态工具** 读写列表。

**核心配置一览：**

| 配置项 | 值 |
|--------|-----|
| `session_state` | `{"shopping_list": []}` |
| `enable_agentic_state` | `True`（Team 与 shopping_agent） |
| `db` | `SqliteDb(tmp/agents.db)` |

## 运行机制与因果链

状态在 run 间持久化（同 session）；`get_session_state()` 打印最终 dict。

## System Prompt 组装

`<session_state>` 段在 `add_session_state_to_context` 时进入 system（`agno/team/_messages.py` `_get_formatted_session_state_for_system_message`）。

## Mermaid 流程图

```mermaid
flowchart TD
    S["session_state"] --> A["【关键】agentic 工具读写"]
    A --> D["DB 持久会话"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/team/_messages.py` | session_state 注入 |
