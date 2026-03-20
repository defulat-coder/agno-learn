# openai_agents_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/openai_agents_instantiation.py`

## 概述

本示例测量 **OpenAI Agents SDK**（或官方 `openai-agents` 包）中 Agent 构造/注册耗时，由 `PerformanceEval` 包裹（细节见源文件）。

## 核心组件解析

与 `instantiate_agent.py`（纯 Agno）对照阅读，理解不同抽象层开销。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | 基准 |
