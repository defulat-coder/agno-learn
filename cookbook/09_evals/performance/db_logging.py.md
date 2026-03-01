# db_logging.py (performance) — 实现原理分析

> 源文件：`cookbook/09_evals/performance/db_logging.py`

## 概述

本示例展示 **`PerformanceEval`** 的 **PostgreSQL 数据库日志记录**机制：将性能测量结果（运行时、内存统计）持久化到 `eval_runs_cookbook` 表。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `db` | `PostgresDb(db_url=..., eval_table="eval_runs_cookbook")` | PostgreSQL 日志 DB |
| `name` | `"Simple Performance Evaluation"` | 评估名称 |
| `func` | `run_agent` | 被测函数 |
| `num_iterations` | `1` | 1 次迭代 |
| `warmup_runs` | `0` | 无预热 |

## 核心组件解析

### 性能结果 DB 写入

`PerformanceEval.run()` 完成后，调用 `log_eval_run()`（`performance.py:592-610`）：

```python
if self.db:
    eval_input = {
        "num_iterations": self.num_iterations,
        "warmup_runs": self.warmup_runs,
    }
    log_eval_run(
        db=self.db,
        run_id=self.eval_id,
        run_data=self._parse_eval_run_data(),  # avg/min/max/p95 等
        eval_type=EvalType.PERFORMANCE,
        name=self.name,
        evaluated_component_name=self.func.__name__,  # "run_agent"
        agent_id=self.agent_id,  # None（未设置）
        eval_input=eval_input,
    )
```

### _parse_eval_run_data() 写入内容

```python
{
    "result": {
        "avg_run_time": ..., "min_run_time": ..., "max_run_time": ...,
        "std_dev_run_time": ..., "median_run_time": ..., "p95_run_time": ...,
        "avg_memory_usage": ..., ...
    },
    "runs": [{"runtime": ..., "memory": ...}, ...]  # 每次迭代详情
}
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `run()` L592-610 | DB 日志写入 |
| `agno/eval/performance.py` | `_parse_eval_run_data()` L284 | 结果序列化 |
| `agno/eval/utils.py` | `log_eval_run()` | 通用日志写入 |
| `agno/db/schemas/evals.py` | `EvalType.PERFORMANCE` | 评估类型标识 |
