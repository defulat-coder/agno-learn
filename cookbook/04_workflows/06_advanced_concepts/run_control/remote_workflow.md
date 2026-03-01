# remote_workflow.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/run_control/remote_workflow.py`

## 概述

本示例展示 Agno Workflow **远程 Workflow 执行**：通过 `RemoteWorkflow` 连接到托管在远程服务器（AgentOS）上的 Workflow，以与本地 Workflow 完全相同的 API（`arun(stream=False/True)`）进行调用，实现跨服务器的工作流编排。

**核心 API：**

| API | 说明 |
|-----|------|
| `RemoteWorkflow(base_url, workflow_id)` | 连接远程 Workflow |
| `remote_workflow.arun(input, stream=False)` | 非流式调用 |
| `remote_workflow.arun(input, stream=True, stream_events=True)` | 流式调用 |

## 核心组件解析

```python
from agno.workflow import RemoteWorkflow

# 连接远程 AgentOS 服务
remote_workflow = RemoteWorkflow(
    base_url=os.getenv("AGNO_REMOTE_BASE_URL", "http://localhost:7777"),
    workflow_id=os.getenv("AGNO_REMOTE_WORKFLOW_ID", "qa-workflow"),
)

# 非流式调用（与本地 Workflow 相同 API）
response = await remote_workflow.arun(
    input="Summarize the latest progress in AI coding assistants.",
    stream=False,
)

# 流式调用
async for event in remote_workflow.arun(input="...", stream=True, stream_events=True):
    content = getattr(event, "content", None)
    if content:
        print(content, end="", flush=True)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/__init__.py` | `RemoteWorkflow` | 远程 Workflow 客户端 |
