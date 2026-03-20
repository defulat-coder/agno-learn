# calculator.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Single Tool Call Reliability Evaluation
=======================================

Demonstrates reliability checks for one expected tool call.
"""

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
    response: RunOutput = agent.run("What is 10! (ten factorial)?")
    evaluation = ReliabilityEval(
        name="Tool Call Reliability",
        agent_response=response,
        expected_tool_calls=["factorial"],
    )
    result: Optional[ReliabilityResult] = evaluation.run(print_results=True)
    if result:
        result.assert_passed()


# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    factorial()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/reliability/single_tool_calls/calculator.py`

## 概述

本示例为 **单次工具调用可靠性**：`agent.run("What is 10! (ten factorial)?")`，期望 `expected_tool_calls=["factorial"]`，无 DB。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `ReliabilityEval.name` | `"Tool Call Reliability"` | 命名 |

## 核心组件解析

最小复现：计算器工具是否按预期名称被调用（受模型行为影响）。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/reliability.py` | 比对 tool_calls |
