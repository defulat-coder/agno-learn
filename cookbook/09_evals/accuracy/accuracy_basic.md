# accuracy_basic.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Basic Accuracy Evaluation
=========================

Demonstrates synchronous and asynchronous accuracy evaluations.
"""

import asyncio
from typing import Optional

from agno.agent import Agent
from agno.eval.accuracy import AccuracyEval, AccuracyResult
from agno.models.openai import OpenAIChat
from agno.tools.calculator import CalculatorTools

# ---------------------------------------------------------------------------
# Create Sync Evaluation
# ---------------------------------------------------------------------------
evaluation = AccuracyEval(
    name="Calculator Evaluation",
    model=OpenAIChat(id="o4-mini"),
    agent=Agent(
        model=OpenAIChat(id="gpt-4o"),
        tools=[CalculatorTools()],
    ),
    input="What is 10*5 then to the power of 2? do it step by step",
    expected_output="2500",
    additional_guidelines="Agent output should include the steps and the final answer.",
    num_iterations=1,
)

# ---------------------------------------------------------------------------
# Create Async Evaluation
# ---------------------------------------------------------------------------
async_evaluation = AccuracyEval(
    model=OpenAIChat(id="o4-mini"),
    agent=Agent(
        model=OpenAIChat(id="gpt-4o"),
        tools=[CalculatorTools()],
    ),
    input="What is 10*5 then to the power of 2? do it step by step",
    expected_output="2500",
    additional_guidelines="Agent output should include the steps and the final answer.",
    num_iterations=3,
)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    result: Optional[AccuracyResult] = evaluation.run(print_results=True)
    assert result is not None and result.avg_score >= 8

    async_result: Optional[AccuracyResult] = asyncio.run(
        async_evaluation.arun(print_results=True)
    )
    assert async_result is not None and async_result.avg_score >= 8
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/accuracy/accuracy_basic.py`

## 概述

本示例演示 **`AccuracyEval.run`（同步）与 `arun`（异步）**：计算器 Agent、`o4-mini` 评判、`num_iterations` 在异步版本为 3。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `evaluation` | `num_iterations=1` | 同步单次 |
| `async_evaluation` | `num_iterations=3` | 异步多次迭代 |
| `expected_output` | `2500` | 期望最终数值 |

## 核心组件解析

`asyncio.run(async_evaluation.arun(...))` 验证异步评测路径与指标聚合。

## System Prompt 组装

被测 Agent 未设 `instructions`，仅有默认 markdown 等；评判端见 `accuracy.py`。

## 完整 API 请求

同步与异步两套 `chat.completions` / `achat` 路径（以各 `OpenAIChat` 方法为准）。

## Mermaid 流程图

```mermaid
flowchart TD
    S["evaluation.run"] --> R["结果"]
    A["async_evaluation.arun"] --> R2["多迭代 avg_score"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/accuracy.py` | `run` / `arun` |
