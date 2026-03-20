# 01_team_always_learn.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Team Learning: Always Mode
==========================
Set learning=True on a Team to enable automatic learning.

The team automatically captures:
- User profile: name, role, preferences
- User memory: observations, context, patterns

Extraction runs in parallel after each response.
This is the simplest way to add learning to a team.
"""

from agno.agent import Agent
from agno.db.postgres import PostgresDb
from agno.models.openai import OpenAIResponses
from agno.team import Team

db = PostgresDb(db_url="postgresql+psycopg://ai:ai@localhost:5532/ai")


# ---------------------------------------------------------------------------
# Create Members
# ---------------------------------------------------------------------------
researcher = Agent(
    name="Researcher",
    model=OpenAIResponses(id="gpt-5.2"),
    role="Research topics and provide detailed information.",
)

writer = Agent(
    name="Writer",
    model=OpenAIResponses(id="gpt-5.2"),
    role="Write clear, concise content based on research.",
)


# ---------------------------------------------------------------------------
# Create Team
# ---------------------------------------------------------------------------
team = Team(
    name="Research Team",
    model=OpenAIResponses(id="gpt-5.2"),
    members=[researcher, writer],
    db=db,
    learning=True,
    markdown=True,
    show_members_responses=True,
)


# ---------------------------------------------------------------------------
# Run Demo
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    user_id = "alice@example.com"

    # Session 1: Share information naturally
    print("\n" + "=" * 60)
    print("SESSION 1: Team learns about the user automatically")
    print("=" * 60 + "\n")

    team.print_response(
        "Hi! I'm Alice, a machine learning engineer. "
        "I prefer technical explanations with code examples. "
        "Can you explain how attention mechanisms work?",
        user_id=user_id,
        session_id="session_1",
        stream=True,
    )

    lm = team.learning_machine
    print("\n--- Learned Profile ---")
    lm.user_profile_store.print(user_id=user_id)
    print("\n--- Learned Memories ---")
    lm.user_memory_store.print(user_id=user_id)

    # Session 2: New session - team remembers
    print("\n" + "=" * 60)
    print("SESSION 2: Team remembers across sessions")
    print("=" * 60 + "\n")

    team.print_response(
        "What do you know about me? And can you explain transformers?",
        user_id=user_id,
        session_id="session_2",
        stream=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/03_teams/12_learning/01_team_always_learn.py`

## 概述

本示例展示 **`learning=True` 简写**：在 Team 上开启自动学习，`LearningMachine` 使用默认配置在每次响应后并行抽取用户画像与记忆，并写入 `PostgresDb`。

**核心配置一览：**

| 配置项 | 值 |
|--------|-----|
| `db` | `PostgresDb(postgresql+psycopg://ai:ai@localhost:5532/ai)` |
| `learning` | `True` |
| `members` | Researcher + Writer |
| `markdown` | `True` |
| `show_members_responses` | `True` |

### 运行机制与因果链

1. **路径**：`print_response` 带 `user_id`/`session_id` → Team 跑对话 → 学习管线异步提取 → `team.learning_machine` 各 store。
2. **状态**：Postgres 持久化；`session_1` 与 `session_2` 展示跨会话回忆。
3. **定位**：最简 **Team 学习** 入口，无自定义 `LearningMachine(...)`。

## System Prompt 组装

学习机制会额外向运行上下文注入与 learn 相关的工具/说明（以 `agno/learn` 实现为准）；本文件未自定义 `instructions`，队长为默认 Team 模板。

## 完整 API 请求

```python
client.responses.create(model="gpt-5.2", input=formatted_input)
```

## Mermaid 流程图

```mermaid
flowchart TD
    P["print_response"] --> L["【关键】learning=True 提取管线"]
    L --> S["user_profile_store / user_memory_store"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/team/team.py` | `learning` 字段 |
| `agno/learn/` | `LearningMachine` 与 stores |
