# db_logging.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Performance Evaluation with Database Logging
============================================

Demonstrates storing performance evaluation results in PostgreSQL.
"""

from agno.agent import Agent
from agno.db.postgres.postgres import PostgresDb
from agno.eval.performance import PerformanceEval
from agno.models.openai import OpenAIChat


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
def run_agent():
    agent = Agent(
        model=OpenAIChat(id="gpt-5.2"),
        system_message="Be concise, reply with one sentence.",
    )
    response = agent.run("What is the capital of France?")
    print(response.content)
    return response


# ---------------------------------------------------------------------------
# Create Database
# ---------------------------------------------------------------------------
db_url = "postgresql+psycopg://ai:ai@localhost:5432/ai"
db = PostgresDb(db_url=db_url, eval_table="eval_runs_cookbook")

# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
simple_response_perf = PerformanceEval(
    db=db,
    name="Simple Performance Evaluation",
    func=run_agent,
    num_iterations=1,
    warmup_runs=0,
)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    simple_response_perf.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/db_logging.py`

## 概述

本示例在 **`PerformanceEval` 上配置 `PostgresDb`**（`eval_table="eval_runs_cookbook"`，端口 **5432**），将性能评测运行写入数据库。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `simple_response_perf.db` | `db` | 持久化 |
| `func` | `run_agent`：单轮法国首都问答 | 与 `simple_response.py` 同构 |

### 还原 system_message

```text
Be concise, reply with one sentence.
```

## 核心组件解析

副作用：每次 `run` 写入 eval 表，便于对比迭代耗时与历史。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | DB 集成 |
| `agno/db/postgres` | `eval_table` |
