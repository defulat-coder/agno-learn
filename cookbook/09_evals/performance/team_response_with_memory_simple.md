# team_response_with_memory_simple.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/team_response_with_memory_simple.py`

## 概述

本示例用 **`PerformanceEval`** 压测 **带记忆与工具的 Team**：`PostgresDb`、`weather_agent` 等成员、`get_weather` 工具，随机城市与用户输入（脚本后半部分定义异步/并发与 `num_iterations`，以源文件为准）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `db` | Postgres 5532 | 团队与会话 |
| `Team` + `Agent` | 天气多智能体场景 | 见源码成员定义 |

## 核心组件解析

关注 Team 调度 + DB 会话 + 工具调用链路的综合延迟；具体迭代次数与 warmup 以 `.py` 底部 `PerformanceEval` 为准。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/team/team.py` | `run` / memory |
| `agno/eval/performance.py` | 基准 |
