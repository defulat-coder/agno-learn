# websocket_client.md — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/background_execution/websocket_client.py`

## 概述

本示例展示 Agno Workflow **WebSocket 客户端**的实现：通过 `websockets` 库连接到 WebSocket 服务端，发送 Token 认证、启动 Workflow、监听流式事件，并将各类 Workflow 事件（`StepStarted`、`RunContent`、`WorkflowCompleted` 等）格式化为 Rich 面板展示，适用于与 `websocket_server.py` 配套使用的交互式 CLI 客户端。

**核心交互命令：**

| 命令 | 说明 |
|------|------|
| `auth` | 发送 Token 认证 |
| `start <message>` | 启动 Workflow |
| `ping` | 心跳检测 |
| `quit` | 断开连接 |

## 核心组件解析

### 连接与认证

```python
class WorkflowWebSocketClient:
    async def connect(self):
        self.websocket = await websockets.connect(self.server_url)
        if self.auth_token:
            await self.authenticate()

    async def authenticate(self, token=None):
        auth_message = {"action": "authenticate", "token": auth_token}
        await self.websocket.send(json.dumps(auth_message))
```

### 流式事件监听

```python
async def listen_for_events(self):
    async for message in self.websocket:
        event_data = json.loads(message)
        panel = self.format_event(event_data)   # 格式化 Rich 面板
        if panel:
            self.console.print(panel)
```

### 事件类型映射

| 事件 | 说明 |
|------|------|
| `WorkflowStarted` | Workflow 开始 |
| `StepStarted/StepCompleted` | 步骤开始/完成 |
| `RunContent` | 流式内容块（累积显示） |
| `WorkflowCompleted` | Workflow 完成 |

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `websockets` | `websockets.connect()` | 建立 WebSocket 连接 |
| 本文件 | `WorkflowWebSocketClient` | 客户端逻辑封装 |
