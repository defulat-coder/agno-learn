# crewai_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/crewai_instantiation.py`

## 概述

本示例用 **`PerformanceEval`** 测量 **CrewAI** 框架下 Agent/Crew 的实例化耗时，用于与 Agno 对照（具体类名与 `func` 以源文件为准）。

## 核心组件解析

不包含 Agno `get_system_message`；属于跨框架微基准。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | `PerformanceEval` |
