# calculator.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Multiple Tool Call Reliability Evaluation
=========================================

Demonstrates reliability checks for multiple expected tool calls.
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
def multiply_and_exponentiate():
    agent = Agent(
        model=OpenAIChat(id="gpt-5.2"),
        tools=[CalculatorTools()],
    )
    response: RunOutput = agent.run(
        "What is 10*5 then to the power of 2? do it step by step"
    )
    evaluation = ReliabilityEval(
        name="Tool Calls Reliability",
        agent_response=response,
        expected_tool_calls=["multiply", "exponentiate"],
    )
    result: Optional[ReliabilityResult] = evaluation.run(print_results=True)
    if result:
        result.assert_passed()


# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    multiply_and_exponentiate()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/reliability/multiple_tool_calls/calculator.py`

## 概述

本示例期望 **连续两步工具**：`multiply` 与 `exponentiate`，输入为「10*5 再平方」类逐步指令。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `expected_tool_calls` | `["multiply", "exponentiate"]` | 顺序/集合语义以 `ReliabilityEval` 实现为准 |

## 核心组件解析

用于抓「少调一次工具」或「顺序错误」类回归。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/calculator` | 工具名定义 |
