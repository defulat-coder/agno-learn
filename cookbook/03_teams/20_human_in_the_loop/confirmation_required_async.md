# confirmation_required_async.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/confirmation_required_async.py`

## 概述

`confirmation_required.py` 的**异步模式变体**。使用 `await team.arun()` 和 `await team.acontinue_run(run_response)`，其余确认逻辑完全相同。

**关键差异：**

```python
# 异步首次运行
run_response = await team.arun("Deploy app v2.0", session_id=session_id)

if run_response.is_paused:
    for requirement in run_response.active_requirements:
        if requirement.needs_confirmation:
            requirement.confirm()  # 或 requirement.reject()

    # 异步恢复
    run_response = await team.acontinue_run(run_response)
```

适合在 FastAPI、asyncio 服务中集成 HITL 流程。
