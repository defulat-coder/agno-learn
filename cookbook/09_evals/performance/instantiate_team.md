# instantiate_team.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Team Instantiation Performance Evaluation
=========================================

Demonstrates measuring team instantiation performance.
"""

from agno.agent import Agent
from agno.eval.performance import PerformanceEval
from agno.models.openai import OpenAIChat
from agno.team.team import Team

# ---------------------------------------------------------------------------
# Create Team Member
# ---------------------------------------------------------------------------
team_member = Agent(model=OpenAIChat(id="gpt-4o"))


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
def instantiate_team():
    return Team(members=[team_member])


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
instantiation_perf = PerformanceEval(
    name="Instantiation Performance Team", func=instantiate_team, num_iterations=1000
)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    instantiation_perf.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/instantiate_team.py`

## 概述

本示例测量 **`Team(members=[team_member])` 构造** 耗时，`num_iterations=1000`，无 `run`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `team_member` | `Agent(model=OpenAIChat gpt-4o)` | 单成员 |

## 核心组件解析

对比 `instantiate_agent`：Team 协调器与成员图初始化成本。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/team/team.py` | `Team.__init__` |
