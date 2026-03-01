# 03_team_session_metrics.py — 实现原理分析

> 源文件：`cookbook/03_teams/22_metrics/03_team_session_metrics.py`

## 概述

本示例展示 **会话级累计指标**：同一 `session_id` 下执行多次 `team.run()`，`team.get_session_metrics()` 返回跨所有 run 累计的 token 用量和耗时，适合追踪整个对话会话的资源消耗。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `session_id` | `"team_session_metrics_demo"` | 固定 session 以累计指标 |
| `db` | `PostgresDb` | 跨 run 持久化指标 |
| `team.get_session_metrics()` | 无参（用已绑定的 session_id） | 获取累计指标 |

## 核心组件解析

### 指标累计逻辑

```python
# 第1次 run
run_output_1 = team.run("What is the capital of Japan?")
pprint(run_output_1.metrics)  # 仅本次 run 的指标

# 第2次 run（同 session）
run_output_2 = team.run("What about South Korea?")
pprint(run_output_2.metrics)  # 仅本次 run 的指标

# 会话级（两次累计）
session_metrics = team.get_session_metrics()
pprint(session_metrics)  # total_tokens = run1.tokens + run2.tokens
```

### 使用场景

- 成本追踪：统计单个用户会话消耗的总 token
- 性能分析：识别高 token 消耗的会话
- 账单核算：按 session 计费

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/team/team.py` | `get_session_metrics()` | 会话累计指标 |
| `agno/db/postgres.py` | `PostgresDb` | 跨 run 持久化 |
