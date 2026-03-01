# change_state_on_run.py — 实现原理分析

> 源文件：`cookbook/03_teams/21_state/change_state_on_run.py`

## 概述

本示例展示 **每次 `run()` 传入不同 `session_state`** 服务不同用户：同一个 Team 实例，通过 `session_id` + `user_id` + `session_state` 的组合，为不同用户（John、Jane）维护独立的会话状态。同一 session 的后续 run 无需重复传入 `session_state`，状态从 `InMemoryDb` 自动读取。

**核心配置一览：**

| run 调用 | session_id | user_id | session_state |
|---------|-----------|---------|--------------|
| 第1次 | user_1_session_1 | user_1 | {user_name:"John", age:30} |
| 第2次 | user_1_session_1 | user_1 | 无（从 db 读取） |
| 第3次 | user_2_session_1 | user_2 | {user_name:"Jane", age:25} |

## 核心组件解析

### 多用户状态隔离

```python
# 用户1的第一轮：设置状态
team.print_response(
    "What is my name?",
    session_id="user_1_session_1",
    user_id="user_1",
    session_state={"user_name": "John", "age": 30},
)

# 用户1的第二轮：状态自动恢复
team.print_response(
    "How old am I?",
    session_id="user_1_session_1",
    user_id="user_1",
    # 无需再传 session_state，InMemoryDb 自动读取
)
```

### 状态与 session_id 绑定

`session_state` 存储在 `InMemoryDb` 中，key 为 `session_id`。不同 `session_id` 的状态完全隔离，同一 `session_id` 的多次 run 共享同一状态。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/team/team.py` | `session_state`（run 参数） | 每次运行的状态 |
| `agno/db/in_memory.py` | `InMemoryDb` | 内存会话存储 |
