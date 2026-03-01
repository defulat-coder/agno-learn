# websocket_server.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/background_execution/websocket_server.py`

## 概述

本示例展示 Agno Workflow **通过 WebSocket + FastAPI 实现后台流式推送**：使用 `workflow.arun(background=True, websocket=websocket, stream=True, stream_events=True)` 在后台执行 Workflow，将事件实时推送给 WebSocket 客户端。服务端还实现了连接认证（Token 校验）、连接管理等生产级特性。

**核心配置：**

| 参数 | 说明 |
|------|------|
| `background=True` | 后台异步执行 |
| `websocket=websocket` | 事件实时推送的 WebSocket 连接 |
| `stream_events=True` | 推送所有细粒度事件 |
| `stream=True` | 启用流式输出 |

## 核心组件解析

### FastAPI WebSocket + 后台 Workflow

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    # ... 认证逻辑 ...

async def handle_start_workflow(websocket: WebSocket, message_data: dict):
    workflow = Workflow(
        steps=[
            Step(name="hackernews_research", agent=hackernews_agent),
            Step(name="web_search", agent=search_agent),
        ],
        db=SqliteDb(db_file="tmp/workflow_bg.db"),
    )

    result = await workflow.arun(
        input=message,
        session_id=session_id,
        stream=True,
        stream_events=True,
        background=True,
        websocket=websocket,   # 直接推送到 WebSocket 客户端
    )
```

### 认证流程

```
客户端 -> authenticate(token) -> 服务端验证 -> authenticated/auth_error
客户端 -> start-workflow(message) -> 启动后台 Workflow -> 流式事件推送
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.arun(websocket=...)` | 后台执行并推送 WebSocket 事件 |
| FastAPI `WebSocket` | `WebSocket.send_text()` | 向客户端推送 JSON 事件 |
