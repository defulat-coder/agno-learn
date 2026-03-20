# calculator.py — 实现原理分析

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
