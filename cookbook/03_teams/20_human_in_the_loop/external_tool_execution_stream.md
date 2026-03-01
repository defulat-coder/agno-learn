# external_tool_execution_stream.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/external_tool_execution_stream.py`

## 概述

`external_tool_execution.py` 的**流式模式变体**：`@tool(external_execution=True)` 工具在流式事件中触发暂停，通过 `TeamRunPausedEvent` 获取工具参数，外部执行后调用 `requirement.set_external_execution_result(result)`，再 `team.continue_run()` 恢复。
