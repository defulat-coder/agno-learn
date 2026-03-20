# langgraph_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/langgraph_instantiation.py`

## 概述

本示例用 **`PerformanceEval`** 测量 **LangGraph** 图或节点构建/编译的实例化耗时（具体 API 以源文件为准），用于横向对比。

## 核心组件解析

纯框架构造路径，不涉及 Agno Agent。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | 计时封装 |
