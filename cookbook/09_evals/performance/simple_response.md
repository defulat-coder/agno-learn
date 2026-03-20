# simple_response.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Simple Response Performance Evaluation
======================================

Demonstrates baseline response performance for a single prompt.
"""

from agno.agent import Agent
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
    print(f"Agent response: {response.content}")

    return response


# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
simple_response_perf = PerformanceEval(
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

> 源文件：`cookbook/09_evals/performance/simple_response.py`

## 概述

本示例为 **端到端单次响应** 基线：`run_agent` 内创建 Agent、`run` 一问一答，`num_iterations=1`，`warmup_runs=0`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `system_message` | 短答一句 | 见下 |
| `model` | `gpt-5.2` | Chat |

### 还原 system_message

```text
Be concise, reply with one sentence.
```

## 核心组件解析

作为与其它性能用例（memory、storage、team）对比的基准延迟。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | 计时 |
