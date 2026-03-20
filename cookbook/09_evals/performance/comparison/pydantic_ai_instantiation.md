# pydantic_ai_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/pydantic_ai_instantiation.py`

## 概述

本示例测量 **PydanticAI** Agent 实例化耗时，`PerformanceEval` 统一计时（见源文件中的 `func` 与迭代次数）。

## 核心组件解析

微基准；不经过 Agno 消息组装管线。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | `PerformanceEval` |
