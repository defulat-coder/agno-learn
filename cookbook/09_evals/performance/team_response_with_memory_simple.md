# team_response_with_memory_simple.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Simple Team Memory Performance Evaluation
=========================================

Demonstrates team response performance with memory enabled.
"""

import asyncio
import random

from agno.agent import Agent
from agno.db.postgres import PostgresDb
from agno.eval.performance import PerformanceEval
from agno.models.openai import OpenAIChat
from agno.team.team import Team

# ---------------------------------------------------------------------------
# Create Sample Inputs
# ---------------------------------------------------------------------------
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
# Create Tool
# ---------------------------------------------------------------------------
def get_weather(city: str) -> str:
    return f"The weather in {city} is sunny."


# ---------------------------------------------------------------------------
# Create Team
# ---------------------------------------------------------------------------
weather_agent = Agent(
    id="weather_agent",
    model=OpenAIChat(id="gpt-5.2"),
    role="Weather Agent",
    description="You are a helpful assistant that can answer questions about the weather.",
    instructions="Be concise, reply with one sentence.",
    tools=[get_weather],
    db=db,
    update_memory_on_run=True,
    add_history_to_context=True,
)

team = Team(
    members=[weather_agent],
    model=OpenAIChat(id="gpt-5.2"),
    instructions="Be concise, reply with one sentence.",
    db=db,
    markdown=True,
    update_memory_on_run=True,
    add_history_to_context=True,
)


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
async def run_team():
    random_city = random.choice(cities)
    _ = team.arun(
        input=f"I love {random_city}! What weather can I expect in {random_city}?",
        stream=True,
        stream_events=True,
    )

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

> 源文件：`cookbook/09_evals/performance/team_response_with_memory_simple.py`

## 概述

本示例用 **`PerformanceEval`** 压测 **带记忆与工具的 Team**：`PostgresDb`、`weather_agent` 等成员、`get_weather` 工具，随机城市与用户输入（脚本后半部分定义异步/并发与 `num_iterations`，以源文件为准）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `db` | Postgres 5532 | 团队与会话 |
| `Team` + `Agent` | 天气多智能体场景 | 见源码成员定义 |

## 核心组件解析

关注 Team 调度 + DB 会话 + 工具调用链路的综合延迟；具体迭代次数与 warmup 以 `.py` 底部 `PerformanceEval` 为准。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/team/team.py` | `run` / memory |
| `agno/eval/performance.py` | 基准 |
