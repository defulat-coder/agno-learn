# team_response_with_memory_multi_user.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Multi-User Team Memory Performance Evaluation
=============================================

Demonstrates concurrent team performance across multiple users with memory.
"""

import asyncio
import random
import uuid

from agno.agent import Agent
from agno.db.postgres import PostgresDb
from agno.eval.performance import PerformanceEval
from agno.models.openai import OpenAIChat
from agno.team.team import Team

# ---------------------------------------------------------------------------
# Create Sample Inputs
# ---------------------------------------------------------------------------
users = [
    "abel@example.com",
    "ben@example.com",
    "charlie@example.com",
    "dave@example.com",
    "edward@example.com",
]

cities = [
    "New York",
    "Los Angeles",
    "Chicago",
    "Houston",
    "Miami",
    "San Francisco",
    "Seattle",
    "Boston",
    "Washington D.C.",
    "Atlanta",
    "Denver",
    "Las Vegas",
]

# ---------------------------------------------------------------------------
# Create Database
# ---------------------------------------------------------------------------
db_url = "postgresql+psycopg://ai:ai@localhost:5532/ai"
db = PostgresDb(db_url=db_url)


# ---------------------------------------------------------------------------
# Create Tools
# ---------------------------------------------------------------------------
def get_weather(city: str) -> str:
    return f"The weather in {city} is sunny."


def get_activities(city: str) -> str:
    activities = [
        "hiking",
        "biking",
        "swimming",
        "kayaking",
        "museum visits",
        "shopping",
        "sightseeing",
        "cafe hopping",
        "theater",
        "picnicking",
    ]
    selected_activities = random.sample(activities, k=3)
    return f"The activities in {city} are {', '.join(selected_activities)}."


# ---------------------------------------------------------------------------
# Create Team
# ---------------------------------------------------------------------------
weather_agent = Agent(
    id="weather_agent",
    model=OpenAIChat(id="gpt-5.2"),
    description="You are a helpful assistant that can answer questions about the weather.",
    instructions="Be concise, reply with one sentence.",
    tools=[get_weather],
    db=db,
    update_memory_on_run=True,
    add_history_to_context=True,
)

activities_agent = Agent(
    id="activities_agent",
    model=OpenAIChat(id="gpt-5.2"),
    description="You are a helpful assistant that can answer questions about activities in a city.",
    instructions="Be concise, reply with one sentence.",
    tools=[get_activities],
    db=db,
    update_memory_on_run=True,
    add_history_to_context=True,
)

team = Team(
    members=[weather_agent, activities_agent],
    model=OpenAIChat(id="gpt-5.2"),
    instructions="Be concise, reply with one sentence.",
    db=db,
    update_memory_on_run=True,
    markdown=True,
    add_history_to_context=True,
)


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
async def run_team():
    async def run_team_for_user(user: str):
        random_city = random.choice(cities)
        await team.arun(
            input=f"I love {random_city}! What activities and weather can I expect in {random_city}?",
            user_id=user,
            session_id=f"session_{uuid.uuid4()}",
        )

    tasks = []

    # Run all 5 users concurrently
    for user in users:
        tasks.append(run_team_for_user(user))
    await asyncio.gather(*tasks)

    return "Successfully ran team"


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
team_response_with_memory_impact = PerformanceEval(
    name="Team Memory Impact",
    func=run_team,
    num_iterations=5,
    warmup_runs=0,
    measure_runtime=False,
    debug_mode=True,
    memory_growth_tracking=True,
    top_n_memory_allocations=10,
)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    asyncio.run(
        team_response_with_memory_impact.arun(print_results=True, print_summary=True)
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/team_response_with_memory_multi_user.py`

## 概述

本示例测量 **多用户并发场景下 Team + 记忆** 的性能：`asyncio`、多 `user_id`、随机城市与 `get_weather` 工具，`PostgresDb` 持久化。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `users` | 5 个示例邮箱 | 用户隔离 |
| `db_url` | `localhost:5532` | Postgres |

## 核心组件解析

与 `team_response_with_memory_simple` 相比强调 **并发/多用户** 下的吞吐与尾延迟（具体协程编排见源文件）。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/team/team.py` | Team 异步运行 |
