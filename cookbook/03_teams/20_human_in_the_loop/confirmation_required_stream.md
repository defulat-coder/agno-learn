# confirmation_required_stream.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/confirmation_required_stream.py`

## 概述

`confirmation_required.py` 的**流式模式变体**。在流式运行中，通过 `isinstance(event, TeamRunPausedEvent)` 检测暂停事件，其余确认/恢复逻辑与非流式版本相同。

**关键差异：**

```python
from agno.run.team import RunPausedEvent as TeamRunPausedEvent

for event in team.run(input, stream=True, stream_events=True):
    if isinstance(event, TeamRunPausedEvent):
        run_response = event.run_response
        # 处理 active_requirements...
        break
    # 其他事件正常处理（打印内容等）

run_response = team.continue_run(run_response)
```

流式模式下需区分成员 Agent 的暂停事件（`AgentRunPausedEvent`）和 Team 级暂停事件（`TeamRunPausedEvent`），使用 `isinstance()` 区分。
