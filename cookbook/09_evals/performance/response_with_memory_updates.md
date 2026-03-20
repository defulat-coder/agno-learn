# response_with_memory_updates.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/response_with_memory_updates.py`

## 概述

本示例测量 **`update_memory_on_run=True`** 时单次 run 的额外开销：`SqliteDb` + 自我介绍类用户输入触发记忆更新路径。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `db` | `SqliteDb(tmp/memory.db)` | 记忆存储 |
| `update_memory_on_run` | `True` | 跑后更新记忆 |

### 还原 system_message

```text
Be concise, reply with one sentence.
```

## 核心组件解析

与 `simple_response` 对比可观察记忆子系统对延迟的影响。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/memory/` | 更新逻辑 |
