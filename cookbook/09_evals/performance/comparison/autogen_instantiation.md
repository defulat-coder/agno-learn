# autogen_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/autogen_instantiation.py`

## 概述

本示例用 **`PerformanceEval`** 包裹 **AutoGen `AssistantAgent` 实例化**（`autogen_agentchat` + `OpenAIChatCompletionClient`），对比 Agno 自身实例化基准。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| 依赖 | `autogen_agentchat`、`autogen_ext` | 第三方 |
| `func` | `instantiate_agent` → `AssistantAgent(...)` | 仅构造 |

## 核心组件解析

不测 Agno `Agent`，测竞品框架冷启动成本；`num_iterations` 见文件末尾。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | 通用计时器 |
