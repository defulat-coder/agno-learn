# memory.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
This recipe shows how to use personalized memories and summaries in an agent.
Steps:
1. Run: `./cookbook/scripts/run_pgvector.sh` to start a postgres container with pgvector
2. Run: `uv pip install openai sqlalchemy 'psycopg[binary]' pgvector` to install the dependencies
3. Run: `python cookbook/agents/personalized_memories_and_summaries.py` to run the agent
"""

from agno.agent import Agent
from agno.db.postgres import PostgresDb
from agno.models.openai import OpenAIResponses
from rich.pretty import pprint

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

# Setup the database
db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"
db = PostgresDb(db_url=db_url)

agent = Agent(
    model=OpenAIResponses(id="gpt-4o"),
    user_id="test_user",
    session_id="test_session",
    # Pass the database to the Agent
    db=db,
    # Enable user memories
    update_memory_on_run=True,
    # Enable session summaries
    enable_session_summaries=True,
    # Show debug logs so, you can see the memory being created
)

# -*- Share personal information
agent.print_response("My name is john billings?", stream=True)
# -*- Print memories and summary
if agent.db:
    pprint(agent.get_user_memories(user_id="test_user"))
    pprint(
        agent.get_session(session_id="test_session").summary  # type: ignore
    )

# -*- Share personal information
agent.print_response("I live in nyc?", stream=True)
# -*- Print memories
if agent.db:
    pprint(agent.get_user_memories(user_id="test_user"))
    pprint(
        agent.get_session(session_id="test_session").summary  # type: ignore
    )

# -*- Share personal information
agent.print_response("I'm going to a concert tomorrow?", stream=True)
# -*- Print memories
if agent.db:
    pprint(agent.get_user_memories(user_id="test_user"))
    pprint(
        agent.get_session(session_id="test_session").summary  # type: ignore
    )

# Ask about the conversation
agent.print_response(
    "What have we been talking about, do you know my name?", stream=True
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/openai/responses/memory.py`

## 概述

本示例展示 Agno 的 **`PostgresDb` + `update_memory_on_run` + `enable_session_summaries`** 机制：在 Responses 模型上持久化用户记忆与会话摘要，并多轮更新。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(id="gpt-4o")` | Responses |
| `user_id` | `"test_user"` | 用户维度 |
| `session_id` | `"test_session"` | 会话维度 |
| `db` | `PostgresDb(...)` | 持久化 |
| `update_memory_on_run` | `True` | 运行后更新记忆 |
| `enable_session_summaries` | `True` | 会话摘要 |

## 运行机制与因果链

1. **路径**：每轮 `print_response` 后记忆管理器更新；后续问题可带记忆与摘要进 context。
2. **状态**：**写入** PG；`get_user_memories`/`get_session` 读取演示。
3. **分支**：关闭 `update_memory_on_run` 则不写入长期记忆。
4. **定位**：与 `db.py`（仅历史）相比，本文件强调 **用户记忆 + 摘要**。

## System Prompt 组装

当 `add_memories_to_context` 等开启时，`#3.3.9` 段可能注入 memories（见 `_messages.py`）。本文件未显式设置该项，以运行时为准。

## Mermaid 流程图

```mermaid
flowchart TD
    A["用户陈述"] --> B["【关键】update_memory_on_run"]
    B --> C["PostgresDb"]
    C --> D["后续 run 可读取记忆"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/_messages.py` | memories 段 ~L286 起 | 记忆注入 system |
| `agno/db/postgres.py` | `PostgresDb` | 存储 |
