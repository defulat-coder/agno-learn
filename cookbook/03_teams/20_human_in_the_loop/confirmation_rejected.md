# confirmation_rejected.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/confirmation_rejected.py`

## 概述

展示工具确认被**拒绝**后的行为：调用 `requirement.reject(note="User declined")` 后，`team.continue_run()` 让 Agent 感知到工具被拒绝，Agent 会据此调整响应（通常告知用户操作被取消）。

**关键点：**

```python
requirement.reject(note="User declined to approve this operation")
run_response = team.continue_run(run_response)
# Agent 的最终响应会说明操作未执行（因为用户拒绝）
```
