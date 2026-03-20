# ai_news.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Team Reliability Evaluation for News Search
===========================================

Demonstrates tool-call reliability checks for a team workflow.
"""

from typing import Optional

from agno.agent import Agent
from agno.eval.reliability import ReliabilityEval, ReliabilityResult
from agno.models.openai import OpenAIChat
from agno.run.team import TeamRunOutput
from agno.team.team import Team
from agno.tools.websearch import WebSearchTools

# ---------------------------------------------------------------------------
# Create Team
# ---------------------------------------------------------------------------
team_member = Agent(
    name="News Searcher",
    model=OpenAIChat("gpt-4o"),
    role="Searches the web for the latest news.",
    tools=[WebSearchTools(enable_news=True)],
)
team = Team(
    name="News Research Team",
    model=OpenAIChat("gpt-4o"),
    members=[team_member],
    markdown=True,
    show_members_responses=True,
)
expected_tool_calls = [
    "delegate_task_to_member",
    "search_news",
]


# ---------------------------------------------------------------------------
# Create Evaluation Function
# ---------------------------------------------------------------------------
def evaluate_team_reliability():
    response: TeamRunOutput = team.run("What is the latest news on AI?")
    evaluation = ReliabilityEval(
        name="Team Reliability Evaluation",
        team_response=response,
        expected_tool_calls=expected_tool_calls,
    )
    result: Optional[ReliabilityResult] = evaluation.run(print_results=True)
    if result:
        result.assert_passed()


# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    evaluate_team_reliability()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/reliability/team/ai_news.py`

## 概述

本示例对 **`Team.run` 的 `TeamRunOutput`** 做可靠性检查：期望出现 **`delegate_task_to_member`** 与 **`search_news`**（`WebSearchTools(enable_news=True)`）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `ReliabilityEval.team_response` | `TeamRunOutput` | 非单 Agent |
| `expected_tool_calls` | 见源码列表 | Team 编排 + 新闻搜索 |

## 核心组件解析

验证 Team 级工具追踪字段与单 Agent `RunOutput` 的差异处理（见 `agno/eval/reliability.py`）。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/run/team.py` | `TeamRunOutput` |
| `agno/eval/reliability.py` | `team_response` |
