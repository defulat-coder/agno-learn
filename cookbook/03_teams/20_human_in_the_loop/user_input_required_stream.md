# user_input_required_stream.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/user_input_required_stream.py`

## 概述

`user_input_required.py` 的**流式模式变体**：`@tool(requires_user_input=True)` 工具在流式事件中触发暂停，检测 `TeamRunPausedEvent`，收集用户输入后调用 `requirement.provide_user_input(values)`，再 `team.continue_run()` 恢复流式输出。
