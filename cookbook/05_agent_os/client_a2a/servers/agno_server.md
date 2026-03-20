# agno_server.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Agno AgentOS A2A Server for testing A2AClient.

This server uses Agno's AgentOS to create an A2A-compatible
agent that can be tested with A2AClient.

Prerequisites:
    export OPENAI_API_KEY=your_key

Usage:
    python cookbook/06_agent_os/client_a2a/servers/agno_server.py

The server will start at http://localhost:7003
"""

from agno.agent.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.models.openai import OpenAIChat
from agno.os import AgentOS

# ---------------------------------------------------------------------------
# Create Example
# ---------------------------------------------------------------------------

db = SqliteDb(db_file="tmp/agent.db")
chat_agent = Agent(
    name="basic-agent",
    model=OpenAIChat(id="gpt-5.2"),
    id="basic-agent",
    db=db,
    description="A helpful AI assistant that provides thoughtful answers.",
    instructions="You are a helpful AI assistant.",
    add_datetime_to_context=True,
    add_history_to_context=True,
    markdown=True,
)

agent_os = AgentOS(
    agents=[chat_agent],
    a2a_interface=True,
)
app = agent_os.get_app()


# ---------------------------------------------------------------------------
# Run Example
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    agent_os.serve(app="agno_server:app", reload=True, port=7003)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/05_agent_os/client_a2a/servers/agno_server.py`

## 概述

**测试用 AgentOS A2A 服务**：**`chat_agent`** **`id="basic-agent"`**，**`OpenAIChat(gpt-5.2)`**，**`description` + `instructions`**，**`a2a_interface=True`**，端口 **7003**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `description` | 长句 assistant 描述 | `# 3.3.1` |
| `instructions` | `"You are a helpful AI assistant."` | `# 3.3.3` |
| `add_datetime_to_context` | True | 时间 |
| `add_history_to_context` | True | 历史 |
| `markdown` | True | markdown |

## System Prompt 组装

### 还原后的完整 System 文本

```text
A helpful AI assistant that provides thoughtful answers...
```
（`description` 全文见源 L30）

```text
You are a helpful AI assistant.

```

```text
Use markdown to format your answers.

```

```text
The current time is <运行时>.
```

## 完整 API 请求

`OpenAIChat` → `chat.completions.create`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["A2A 请求"] --> B["Agent.run"]
    B --> C["OpenAIChat.invoke"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/os` | `a2a_interface=True` |
