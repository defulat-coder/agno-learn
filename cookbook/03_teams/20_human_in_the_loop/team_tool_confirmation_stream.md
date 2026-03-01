# team_tool_confirmation_stream.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/team_tool_confirmation_stream.py`

## 概述

`team_tool_confirmation.py` 的**流式模式变体**：Team Leader 自身挂载的 `@tool(requires_confirmation=True)` 工具在流式运行中触发暂停，通过 `TeamRunPausedEvent` 检测，处理方式同其他流式 HITL 变体。
