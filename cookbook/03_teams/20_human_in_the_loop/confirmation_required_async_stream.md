# confirmation_required_async_stream.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/confirmation_required_async_stream.py`

## 概述

`confirmation_required.py` 的**异步流式模式变体**：使用 `async for event in team.arun(stream=True, stream_events=True)` 迭代事件，检测 `TeamRunPausedEvent` 后处理确认，再调用 `await team.acontinue_run()` 恢复。结合了异步和流式两种能力。
