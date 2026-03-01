# overwrite_stored_session_state.py — 实现原理分析

> 源文件：`cookbook/03_teams/21_state/overwrite_stored_session_state.py`

## 概述

本示例展示 **`overwrite_db_session_state=True`**：每次 `run()` 传入新 `session_state` 时，直接覆盖数据库中已存储的旧状态，而非合并。这允许每次 run 完全重置状态，适合无状态 API 场景（每次请求携带完整状态）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `overwrite_db_session_state` | `True` | run 时的 session_state 覆盖 db 中的存储 |
| `add_session_state_to_context` | `True` | 状态注入上下文 |

## 核心组件解析

### 覆盖 vs 合并

```python
# 第1次 run：状态 = {"shopping_list": ["Potatoes"]}
team.print_response(
    "...",
    session_state={"shopping_list": ["Potatoes"]},
)
print(team.get_session_state())  # {"shopping_list": ["Potatoes"]}

# 第2次 run（overwrite=True）：状态完全替换为 {"secret_number": 43}
team.print_response(
    "...",
    session_state={"secret_number": 43},
)
print(team.get_session_state())  # {"secret_number": 43}
# shopping_list 消失了！
```

### `overwrite_db_session_state=False`（默认）的行为

若不覆盖，第2次 run 后状态会合并：`{"shopping_list": ["Potatoes"], "secret_number": 43}`。

### 适用场景

- **API 服务**：每次 HTTP 请求携带完整 `session_state`，不依赖服务端持久化
- **测试**：每次 run 从已知状态开始，避免状态污染

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/team/team.py` | `overwrite_db_session_state` | 状态覆盖配置 |
