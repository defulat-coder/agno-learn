# db_logging.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/db_logging.py`

## 概述

本示例在 **`PerformanceEval` 上配置 `PostgresDb`**（`eval_table="eval_runs_cookbook"`，端口 **5432**），将性能评测运行写入数据库。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `simple_response_perf.db` | `db` | 持久化 |
| `func` | `run_agent`：单轮法国首都问答 | 与 `simple_response.py` 同构 |

### 还原 system_message

```text
Be concise, reply with one sentence.
```

## 核心组件解析

副作用：每次 `run` 写入 eval 表，便于对比迭代耗时与历史。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/performance.py` | DB 集成 |
| `agno/db/postgres` | `eval_table` |
