# reliability_async.py — 实现原理分析

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
