# events_replay.md — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/long_running/events_replay.py`

## 概述

本示例展示 AgentOS Workflow WebSocket **已完成 Workflow 的事件回放（Replay）机制**：当 Workflow 已经完成后，客户端通过 `action: "reconnect"` + 任意 `last_event_index` 重连时，服务端会忽略 `last_event_index`，发送 `replay` 通知并从 event_index=0 开始回放**完整**的事件历史，适用于结果审计、事件分析等场景。

**Replay 与 Catch-Up 的区别：**

| 场景 | Workflow 状态 | 通知事件 | 行为 |
|------|------------|---------|------|
| Catch-Up | 运行中 | `catch_up` | 回放 + 继续订阅 |
| Replay | 已完成 | `replay` | 仅回放全部事件 |

## 核心组件解析

### Replay 重连

```python
# Workflow 已完成后重连
await websocket.send(json.dumps({
    "action": "reconnect",
    "run_id": run_id,
    "last_event_index": 10,         # 已完成时此值被忽略，从 0 开始回放
    "workflow_id": "content-creation-workflow",
    "session_id": "replay-test-session",
}))

# 收到 replay 通知
# event_type == "replay": 全量回放开始
#   total_events: 总事件数
# 然后从 event_index=0 接收所有事件，直到 WorkflowCompleted
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| AgentOS WebSocket 服务 | `action: reconnect` | 处理回放逻辑 |
| 事件 `replay` | `total_events` | 提示即将回放的事件总数 |
