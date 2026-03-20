# reliability_async.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Asynchronous Reliability Evaluation
==================================

Demonstrates running reliability checks with asynchronous evaluation.
"""

import asyncio
from typing import Optional

from agno.agent import Agent
from agno.eval.reliability import ReliabilityEval, ReliabilityResult
from agno.models.openai import OpenAIChat
from agno.run.agent import RunOutput
from agno.tools.calculator import CalculatorTools


# ---------------------------------------------------------------------------
# Create Evaluation Function
# ---------------------------------------------------------------------------
def factorial():
    agent = Agent(
        model=OpenAIChat(id="gpt-5.2"),
        tools=[CalculatorTools()],
    )
    response: RunOutput = agent.run("What is 10!?")
    evaluation = ReliabilityEval(
        agent_response=response,
        expected_tool_calls=["factorial"],
    )

    # Run the evaluation calling the arun method.
    result: Optional[ReliabilityResult] = asyncio.run(
        evaluation.arun(print_results=True)
    )
    if result:
        result.assert_passed()


# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    factorial()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/reliability/reliability_async.py`

## 概述

本示例演示 **`ReliabilityEval.arun`**：在同步 `agent.run` 得到 `RunOutput` 后，用 **`asyncio.run(evaluation.arun(...))`** 走异步评测入口，仍期望 `factorial` 工具。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `expected_tool_calls` | `["factorial"]` | 与同步版一致 |

## 核心组件解析

验证可靠性模块在异步 API 下的行为与 `assert_passed()`。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/reliability.py` | `arun` |
