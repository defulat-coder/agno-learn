# team_response_with_memory_multi_user.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/team_response_with_memory_multi_user.py`

## 概述

本示例测量 **多用户并发场景下 Team + 记忆** 的性能：`asyncio`、多 `user_id`、随机城市与 `get_weather` 工具，`PostgresDb` 持久化。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `users` | 5 个示例邮箱 | 用户隔离 |
| `db_url` | `localhost:5532` | Postgres |

## 核心组件解析

与 `team_response_with_memory_simple` 相比强调 **并发/多用户** 下的吞吐与尾延迟（具体协程编排见源文件）。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/team/team.py` | Team 异步运行 |
