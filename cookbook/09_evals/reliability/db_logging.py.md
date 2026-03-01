# db_logging.py (reliability) — 实现原理分析

> 源文件：`cookbook/09_evals/reliability/db_logging.py`

## 概述

本示例展示 **`ReliabilityEval`** 的 **PostgreSQL 数据库日志记录**：将工具调用可靠性评估结果（PASSED/FAILED + 工具名列表）持久化到 `eval_runs` 表。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `db` | `PostgresDb(db_url=..., eval_table="eval_runs")` | PostgreSQL 日志 DB |
| `name` | `"Tool Call Reliability"` | 评估名称 |
| `agent_response` | `agent.run("What is 10!?")` | 被评估响应 |
| `expected_tool_calls` | `["factorial"]` | 期望工具 |

## 核心组件解析

### DB 写入内容

`run()` 完成后（`reliability.py:148-176`）：

```python
if self.db:
    eval_input = {
        "expected_tool_calls": self.expected_tool_calls,  # ["factorial"]
    }
    log_eval_run(
        db=self.db,
        run_id=self.eval_id,
        run_data=asdict(self.result),  # {eval_status, failed_tool_calls, passed_tool_calls}
        eval_type=EvalType.RELIABILITY,
        agent_id=self.agent_response.agent_id,  # 从 RunOutput 提取
        model_id=self.agent_response.model,
        model_provider=self.agent_response.model_provider,
        eval_input=eval_input,
    )
```

### 与 AccuracyEval DB 日志的区别

| 字段 | AccuracyEval | ReliabilityEval |
|------|-------------|-----------------|
| `eval_type` | `EvalType.ACCURACY` | `EvalType.RELIABILITY` |
| `run_data` 内容 | AccuracyResult（分数统计） | ReliabilityResult（工具通过/失败） |
| `eval_input` | guidelines、expected_output 等 | expected_tool_calls |
| 模型信息来源 | `self.agent.model` | `self.agent_response.model`（运行时信息） |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/reliability.py` | `run()` L148-176 | DB 日志写入 |
| `agno/db/schemas/evals.py` | `EvalType.RELIABILITY` | 评估类型枚举 |
| `agno/eval/utils.py` | `log_eval_run()` | 通用日志写入 |
