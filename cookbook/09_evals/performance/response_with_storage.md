# response_with_storage.md — 实现原理分析

> 源文件：`cookbook/09_evals/performance/response_with_storage.py`

## 概述

本示例测量 **`add_history_to_context=True`** 下 **两轮对话**（法国首都 → 人口追问）的延迟：`db=SqliteDb` 持久化会话。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `add_history_to_context` | `True` | 第二轮带历史 |
| `func` 返回 | `response_2.content` | 以第二轮为计时结束点（函数返回内容） |

### 还原 system_message

```text
Be concise, reply with one sentence.
```

## 核心组件解析

覆盖「读历史 + 写消息」的存储路径成本。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/db/sqlite` | 会话存储 |
