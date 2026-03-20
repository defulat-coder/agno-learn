# response_with_storage.md — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Storage-Backed Response Performance Evaluation
==============================================

Demonstrates measuring performance when storage-backed history is enabled.
"""

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.eval.performance import PerformanceEval
from agno.models.openai import OpenAIChat

# ---------------------------------------------------------------------------
# Create Database
# ---------------------------------------------------------------------------
db = SqliteDb(db_file="tmp/storage.db")


# ---------------------------------------------------------------------------
# Create Benchmark Function
# ---------------------------------------------------------------------------
def run_agent():
    agent = Agent(
        model=OpenAIChat(id="gpt-5.2"),
        system_message="Be concise, reply with one sentence.",
        db=db,
        add_history_to_context=True,
    )
    response_1 = agent.run("What is the capital of France?")
    print(response_1.content)

    response_2 = agent.run("How many people live there?")
    print(response_2.content)

    return response_2.content


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
response_with_storage_perf = PerformanceEval(
    name="Storage Performance",
    func=run_agent,
    num_iterations=1,
    warmup_runs=0,
)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    response_with_storage_perf.run(print_results=True, print_summary=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/performance/response_with_storage.py`

## 概述

本示例测量 **`add_history_to_context=True`** 下 **两轮对话**（法国首都 → 人口追问）的延迟：`db=SqliteDb` 持久化会话。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `add_history_to_context` | `True` | 第二轮带历史 |
| `func` 返回 | `response_2.content` | 以第二轮为计时结束点（函数返回内容） |

### 还原 system_message

```text
Be concise, reply with one sentence.
```

## 核心组件解析

覆盖「读历史 + 写消息」的存储路径成本。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/db/sqlite` | 会话存储 |
