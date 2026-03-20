# db_logging.py — 实现原理分析

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
