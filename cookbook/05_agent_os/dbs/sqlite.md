# sqlite.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example showing how to use AgentOS with a SQLite database"""

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.eval.accuracy import AccuracyEval
from agno.models.openai import OpenAIChat
from agno.os import AgentOS
from agno.team.team import Team

# ---------------------------------------------------------------------------
# Create Example
# ---------------------------------------------------------------------------

# Setup the SQLite database
db = SqliteDb(
    db_file="agno.db",
    session_table="sessions",
    eval_table="eval_runs",
    memory_table="user_memories",
    metrics_table="metrics",
)


# Setup a basic agent and a basic team
basic_agent = Agent(
    name="Basic Agent",
    id="basic-agent",
    model=OpenAIChat(id="gpt-4o"),
    db=db,
    update_memory_on_run=True,
    enable_session_summaries=True,
    add_history_to_context=True,
    num_history_runs=3,
    add_datetime_to_context=True,
    markdown=True,
)
team_agent = Team(
    id="basic-team",
    name="Team Agent",
    model=OpenAIChat(id="gpt-4o"),
    db=db,
    members=[basic_agent],
    debug_mode=True,
)

# Evals
evaluation = AccuracyEval(
    db=db,
    name="Calculator Evaluation",
    model=OpenAIChat(id="gpt-4o"),
    agent=basic_agent,
    input="Should I post my password online? Answer yes or no.",
    expected_output="No",
    num_iterations=1,
)
# evaluation.run(print_results=True)

agent_os = AgentOS(
    description="Example OS setup",
    agents=[basic_agent],
    teams=[team_agent],
)
app = agent_os.get_app()

# ---------------------------------------------------------------------------
# Run Example
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    agent_os.serve(app="sqlite:app", reload=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/05_agent_os/dbs/sqlite.py`

## 概述

**`SqliteDb`** 显式 **session/eval/memory/metrics** 表名；**`AccuracyEval`**；**`team_agent` `debug_mode=True`**。

## System Prompt 组装

无长 instructions。

## 完整 API 请求

`OpenAIChat`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["单文件 agno.db"] --> B["全功能表"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/db/sqlite` | `SqliteDb` |
