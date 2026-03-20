# level_5_api.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Level 5: Agentic System (Production API)
==========================================
The most complete level. Production infrastructure for agentic software.
Upgrade from development databases to PostgreSQL + PgVector, add tracing,
and expose everything as an API with AgentOS.

This builds on Level 4 by adding:
- PostgresDb: Production-grade session storage
- PgVector: Production-grade vector search
- AgentOS: FastAPI server with web UI, tracing, and session management

Prerequisites:
    Start PostgreSQL with PgVector:
        ./cookbook/scripts/run_pgvector.sh

    This starts a Postgres container on port 5532 with:
        user=ai, password=ai, database=ai

Run standalone:
    python cookbook/levels_of_agentic_software/level_5_api.py

Run via Agent OS:
    python cookbook/levels_of_agentic_software/run.py
    Then visit https://os.agno.com and select "L5 Coding Agent"

Example prompt:
    "Write a function that validates email addresses using regex"
"""

from pathlib import Path

from agno.agent import Agent
from agno.db.postgres import PostgresDb
from agno.knowledge import Knowledge
from agno.knowledge.embedder.openai import OpenAIEmbedder
from agno.learn import LearnedKnowledgeConfig, LearningMachine, LearningMode
from agno.models.openai import OpenAIResponses
from agno.tools.coding import CodingTools
from agno.tools.reasoning import ReasoningTools
from agno.vectordb.pgvector import PgVector, SearchType

# ---------------------------------------------------------------------------
# Workspace
# ---------------------------------------------------------------------------
WORKSPACE = Path(__file__).parent.joinpath("tmp/code")
WORKSPACE.mkdir(parents=True, exist_ok=True)

# ---------------------------------------------------------------------------
# Production Database
# ---------------------------------------------------------------------------
db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"
db = PostgresDb(db_url=db_url)

# ---------------------------------------------------------------------------
# Knowledge: Static docs (PgVector for production)
# ---------------------------------------------------------------------------
docs_knowledge = Knowledge(
    name="L5 Coding Agent Knowledge",
    vector_db=PgVector(
        db_url=db_url,
        table_name="coding_standards",
        search_type=SearchType.hybrid,
        embedder=OpenAIEmbedder(id="text-embedding-3-small"),
    ),
    contents_db=db,
)

# ---------------------------------------------------------------------------
# Knowledge: Dynamic learnings (PgVector for production)
# ---------------------------------------------------------------------------
learned_knowledge = Knowledge(
    name="L5 Coding Agent Learnings",
    vector_db=PgVector(
        db_url=db_url,
        table_name="coding_learnings",
        search_type=SearchType.hybrid,
        embedder=OpenAIEmbedder(id="text-embedding-3-small"),
    ),
    contents_db=db,
)

# ---------------------------------------------------------------------------
# Agent Instructions
# ---------------------------------------------------------------------------
instructions = """\
You are a production coding agent that learns and improves over time.

## Workflow

1. Search your knowledge and learnings for relevant context
2. Check if the user has preferences you should follow
3. Write code that follows conventions and user preferences
4. Save the code to a file and run it to verify
5. Save any valuable insights or patterns for future use

## Rules

- Always search knowledge and learnings before writing code
- Apply user preferences from memory when writing code
- Save useful coding patterns and insights as learnings
- Follow project conventions from the knowledge base
- No emojis\
"""

# ---------------------------------------------------------------------------
# Create Production Agent
# ---------------------------------------------------------------------------
l5_coding_agent = Agent(
    name="L5 Coding Agent",
    model=OpenAIResponses(id="gpt-5.2"),
    instructions=instructions,
    tools=[
        CodingTools(base_dir=WORKSPACE, all=True),
        ReasoningTools(),
    ],
    knowledge=docs_knowledge,
    search_knowledge=True,
    learning=LearningMachine(
        knowledge=learned_knowledge,
        learned_knowledge=LearnedKnowledgeConfig(
            mode=LearningMode.AGENTIC,
        ),
    ),
    enable_agentic_memory=True,
    db=db,
    add_history_to_context=True,
    num_history_runs=3,
    add_datetime_to_context=True,
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Demo (standalone)
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    l5_coding_agent.print_response(
        "Write a function that validates email addresses using regex. "
        "Save it to email_validator.py and test it with valid and invalid examples.",
        user_id="dev@example.com",
        session_id="production_test",
        stream=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/levels_of_agentic_software/level_5_api.py`

## 概述

Level 5: Agentic System (Production API)

本示例归类：**单 Agent**；模型相关类型：`OpenAIResponses`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | 'L5 Coding Agent' | `Agent(...)` |
| `model` | OpenAIResponses(id='gpt-5.2'…) | `Agent(...)` |
| `instructions` | 'You are a production coding agent that learns and improves over time.\n\n## Workflow\n\n1. Search your knowledge and lear...' | `Agent(...)` |
| `knowledge` | 变量 `docs_knowledge` | `Agent(...)` |
| `search_knowledge` | True | `Agent(...)` |
| `learning` | LearningMachine(…) | `Agent(...)` |
| `enable_agentic_memory` | True | `Agent(...)` |
| `db` | 变量 `db` | `Agent(...)` |
| `add_history_to_context` | True | `Agent(...)` |
| `num_history_runs` | 3 | `Agent(...)` |
| `add_datetime_to_context` | True | `Agent(...)` |
| `markdown` | True | `Agent(...)` |
| （Model 类） | `OpenAIResponses` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ level_5_api.py       │  ──▶  │ Agent → get_run_messages → Model │
└──────────────────────┘         └────────────────────────────────┘
                                          │
                                          ▼
                                  ┌───────────────┐
                                  │ 对应 Model 子类 │
                                  └───────────────┘
```

## 核心组件解析

### 运行机制与因果链

1. **入口**：从模块 `__main__` 或暴露的 `agent` / `team` 调用进入；同步用 `print_response` / `run`，异步用 `aprint_response` / `arun`（若源码中有）。
2. **消息**：默认路径下 system 内容由 `get_system_message()`（`libs/agno/agno/agent/_messages.py` 约 **L106** 起）按分段逻辑拼装；若显式传入 `system_message` 则早退使用该字符串。
3. **模型**：具体 HTTP/SDK 形态以 `libs/agno/agno/models/` 下对应类的 `invoke` / `ainvoke` 为准（勿默认写成单一 `chat.completions`）。
4. **副作用**：若配置 `db`、`knowledge`、`memory`，运行会读写存储；仅以本文件为准对照。

### 与框架的衔接

- **System**：`get_system_message()` 锚点 `agno/agent/_messages.py` **L106+**。
- **运行**：`Agent.print_response` 等入口 `agno/agent/agent.py`（以当前仓库检索为准）。

## System Prompt 组装

| 序号 | 组成部分 | 本文件 | 是否生效 |
|------|---------|--------|---------|
| 1 | `instructions` / `description` 等 | 见核心配置表与源码 | 有赋值则生效 |
| 2 | 默认分段（markdown、时间等） | 取决于 `Agent` 默认与显式参数 | 视参数 |

### 拼装顺序与源码锚点

1. `system_message` 直给 → 使用该内容（见 `_messages.py` 文档字符串分支说明）。
2. 否则默认拼装：`description`、`role`、`instructions`、markdown 附加段等按 `# 3.x` 注释顺序合并。

### 还原后的完整 System 文本

```text
--- instructions ---
You are a production coding agent that learns and improves over time.

## Workflow

1. Search your knowledge and learnings for relevant context
2. Check if the user has preferences you should follow
3. Write code that follows conventions and user preferences
4. Save the code to a file and run it to verify
5. Save any valuable insights or patterns for future use

## Rules

- Always search knowledge and learnings before writing code
- Apply user preferences from memory when writing code
- Save useful coding patterns and insights as learnings
- Follow project conventions from the knowledge base
- No emojis
```

### 段落释义（模型视角）

- 指令与安全边界由 `instructions` / `system_message` 约束；若带 `tools` / `knowledge`，文档中需体现「何时检索/调用」由框架注入的提示段支持。

## 完整 API 请求

```python
# 请以本文件实际 Model 为准打开 libs/agno/agno/models/<厂商>/ 下对应类的 invoke：
# 可能是 chat.completions.create、responses.create、Gemini generate_content 等。
```

> 与上一节 system 文本在同一 run 中组合；`developer`/`system` 角色由适配器转换。

```mermaid
flowchart TD
    Entry["用户入口<br/>`if __name__` / main"] --> Run["【关键】Agent.run / print_response"]
    Run --> Sys["【关键】get_system_message<br/>agno/agent/_messages.py L106+"]
    Sys --> Inv["【关键】Model.invoke / 提供商 API"]
    Inv --> Out["RunOutput / 流式 chunk"]
```

**【关键】节点说明：**

- **print_response / run**：用户可见的同步入口。
- **get_system_message**：系统提示拼装核心。
- **Model.invoke**：对模型提供商的实际请求。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/_messages.py` | `get_system_message()` L106+ |
| `agno/agent/agent.py` | `Agent` 运行与 CLI 输出 |
| `agno/models/` | 各厂商 `Model.invoke` |
