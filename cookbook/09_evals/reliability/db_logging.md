# db_logging.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Reliability Evaluation with Database Logging
============================================

Demonstrates storing reliability evaluation results in PostgreSQL.
"""

from typing import Optional

from agno.agent import Agent
from agno.db.postgres.postgres import PostgresDb
from agno.eval.reliability import ReliabilityEval, ReliabilityResult
from agno.models.openai import OpenAIChat
from agno.run.agent import RunOutput
from agno.tools.calculator import CalculatorTools

# ---------------------------------------------------------------------------
# Create Database
# ---------------------------------------------------------------------------
db_url = "postgresql+psycopg://ai:ai@localhost:5432/ai"
db = PostgresDb(db_url=db_url, eval_table="eval_runs")

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
agent = Agent(
    model=OpenAIChat(id="gpt-5.2"),
    tools=[CalculatorTools()],
)

# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    response: RunOutput = agent.run("What is 10!?")
    evaluation = ReliabilityEval(
        db=db,
        name="Tool Call Reliability",
        agent_response=response,
        expected_tool_calls=["factorial"],
    )
    result: Optional[ReliabilityResult] = evaluation.run(print_results=True)
    if result:
        result.assert_passed()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/reliability/db_logging.py`

## 概述

本示例在 **`ReliabilityEval` 上挂 `PostgresDb`**（`eval_table="eval_runs"`），将 **工具调用可靠性** 判定结果写入 PostgreSQL（端口 **5432**）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `expected_tool_calls` | `["factorial"]` | 期望计算器调用阶乘 |
| `agent_response` | `agent.run("What is 10!?")` 的 `RunOutput` | 含实际 tool_calls |

## 核心组件解析

`ReliabilityEval` 比对 `RunOutput` 中记录的工具名与期望列表；`result.assert_passed()` 失败则抛错。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/reliability.py` | `ReliabilityEval` |
