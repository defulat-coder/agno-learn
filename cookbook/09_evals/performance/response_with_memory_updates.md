# response_with_memory_updates.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Memory Update Performance Evaluation
====================================

Demonstrates measuring performance when memory updates are enabled.
"""

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.eval.performance import PerformanceEval
from agno.models.openai import OpenAIChat

# ---------------------------------------------------------------------------
# Create Database
# ---------------------------------------------------------------------------
db = SqliteDb(db_file="tmp/memory.db")


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
def run_agent():
    agent = Agent(
        model=OpenAIChat(id="gpt-5.2"),
        system_message="Be concise, reply with one sentence.",
        db=db,
        update_memory_on_run=True,
    )

    response = agent.run("My name is Tom! I'm 25 years old and I live in New York.")
    print(f"Agent response: {response.content}")

    return response


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
response_with_memory_updates_perf = PerformanceEval(
    name="Memory Updates Performance",
    func=run_agent,
    num_iterations=5,
    warmup_runs=0,
)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    response_with_memory_updates_perf.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/response_with_memory_updates.py`

## 概述

本示例测量 **`update_memory_on_run=True`** 时单次 run 的额外开销：`SqliteDb` + 自我介绍类用户输入触发记忆更新路径。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `db` | `SqliteDb(tmp/memory.db)` | 记忆存储 |
| `update_memory_on_run` | `True` | 跑后更新记忆 |

### 还原 system_message

```text
Be concise, reply with one sentence.
```

## 核心组件解析

与 `simple_response` 对比可观察记忆子系统对延迟的影响。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/memory/` | 更新逻辑 |
