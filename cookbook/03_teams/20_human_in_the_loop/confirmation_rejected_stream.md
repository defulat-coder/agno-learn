# confirmation_rejected_stream.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/confirmation_rejected_stream.py`

## 概述

`confirmation_rejected.py` 的**流式模式变体**。在流式事件流中检测 `TeamRunPausedEvent`，调用 `requirement.reject()` 后，用 `team.continue_run()` 恢复（流式返回最终拒绝后的响应）。
