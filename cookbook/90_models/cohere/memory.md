# memory.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
This recipe shows how to use personalized memories and summaries in an agent.
Steps:
1. Run: `./cookbook/scripts/run_pgvector.sh` to start a postgres container with pgvector
2. Run: `uv pip install cohere sqlalchemy 'psycopg[binary]' pgvector` to install the dependencies
3. Run: `python cookbook/92_models/cohere/memory.py` to run the agent
"""

from agno.agent.agent import Agent
from agno.db.postgres import PostgresDb
from agno.models.cohere import Cohere

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"
agent = Agent(
    model=Cohere(id="command-a-03-2025"),
    # Store agent sessions in a database
    db=PostgresDb(db_url=db_url),
    update_memory_on_run=True,
    enable_session_summaries=True,
)

# -*- Share personal information
agent.print_response("My name is john billings?", stream=True)

# -*- Share personal information
agent.print_response("I live in nyc?", stream=True)

# -*- Share personal information
agent.print_response("I'm going to a concert tomorrow?", stream=True)

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

> 源文件：`cookbook/90_models/cohere/memory.py`

## 概述

**PostgresDb + Cohere + `update_memory_on_run` + `enable_session_summaries`**，与 anthropic `memory.py` 同模式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cohere(id="command-a-03-2025")` | Cohere |
| `db` | `PostgresDb(...)` | 存储 |
| `update_memory_on_run` | `True` | 记忆更新 |
| `enable_session_summaries` | `True` | 摘要 |

## System Prompt 组装

动态记忆/摘要段落；运行时打印 `get_system_message()`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["多轮对话"] --> B["【关键】memory + summary 写入 db"]
    B --> C["system 注入"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/_messages.py` | `# 3.3.9`–`# 3.3.11` | 记忆与摘要 |
