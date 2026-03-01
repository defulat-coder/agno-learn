# websocket_reconnect.md — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/long_running/websocket_reconnect.py`

## 概述

本示例展示 AgentOS Workflow WebSocket **标准断线重连流程**：Phase 1 启动 Workflow 并记录 `run_id` 和 `last_event_index`，Phase 2 断线后用 `action: "reconnect"` + 记录的 `last_event_index` 重连，服务端发送 `catch_up` 通知并补发漏掉的事件（标记为 MISSED），追赶完成后发送 `subscribed` 并继续推送新事件（标记为 NEW）。

**重连协议关键字段：**

| 字段 | 值 | 说明 |
|------|------|------|
| `action` | `"reconnect"` | 重连动作 |
| `run_id` | 目标 run_id | 要恢复的 Workflow 运行 |
| `last_event_index` | 最后收到的 event_index | 从此位置追赶 |

## 核心组件解析

### 完整重连流程

```python
# Phase 1: 启动 Workflow，记录 run_id 和 last_event_index
for event in initial_events:
    run_id = event.get("run_id")
    last_event_index = event.get("event_index")

# 模拟断线（3 秒），Workflow 继续产生事件

# Phase 2: 重连
await websocket.send(json.dumps({
    "action": "reconnect",
    "run_id": run_id,
    "last_event_index": last_event_index,   # 从此后追赶
    "workflow_id": "content-creation-workflow",
}))

# 收到 catch_up -> MISSED 事件（追赶）-> subscribed -> NEW 事件（实时）
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| AgentOS WebSocket 服务 | `action: reconnect` | 处理重连和事件追赶 |
| 事件 `catch_up` / `subscribed` | - | 追赶状态通知 |
