# calculator.py — 实现原理分析

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
