# disruption_catchup.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/long_running/disruption_catchup.py`

## 概述

本示例展示 AgentOS Workflow WebSocket **断线重连追赶（Catch-Up）机制**：客户端断线后，通过 `action: "reconnect"` + `last_event_index: None` 重新连接到正在运行中的 Workflow，服务端将发送 `catch_up` 通知并从事件 0 开始回放所有已产生的事件，客户端追赶完成后无缝接收后续实时事件。

**重连协议关键字段：**

| 字段 | 值 | 说明 |
|------|------|------|
| `action` | `"reconnect"` | 重连动作 |
| `run_id` | 目标 run_id | 要恢复的 Workflow 运行 |
| `last_event_index` | `None` | 从头追赶所有事件 |

## 核心组件解析

### 断线重连流程

```
Phase 1: 启动 Workflow，接收 3 个事件后断线（模拟网络中断）
Phase 2: 等待 3 秒（Workflow 继续运行）
Phase 3: 重连，发送 last_event_index=None
  -> 收到 catch_up 通知（missed_events、current_event_count）
  -> 回放从 event_index=0 开始的所有事件（无间隙）
  -> 切换为实时订阅（subscribed 通知）
  -> 继续接收新事件
```

### 重连消息格式

```python
await websocket.send(json.dumps({
    "action": "reconnect",
    "run_id": run_id,
    "last_event_index": None,       # None = 从头回放
    "workflow_id": "content-creation-workflow",
    "session_id": "full-catchup-test",
}))
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| AgentOS WebSocket 服务 | `action: reconnect` | 处理重连和事件回放 |
| 事件 `catch_up` | `missed_events` | 提示追赶事件数量 |
| 事件 `subscribed` | `current_event_count` | 追赶完成，切换为实时订阅 |
