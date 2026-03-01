# team_tool_confirmation.py — 实现原理分析

> 源文件：`cookbook/03_teams/20_human_in_the_loop/team_tool_confirmation.py`

## 概述

展示 **Team Leader 自身工具的确认**（而非成员 Agent 的工具）。当 `@tool(requires_confirmation=True)` 工具直接挂在 `Team.tools` 上时，Team Leader 调用时同样触发暂停，与成员工具确认的处理方式完全相同。

**关键差异：**

```python
# 工具直接挂在 Team（不是 Agent）
team = Team(
    members=[...],
    tools=[approve_deployment],  # Team Leader 的工具
    ...
)

# 暂停时 requirement.member_agent_name 为 None（是 Leader 的工具，非成员的）
```

此模式适合需要对 Team Leader 高权限操作（部署、审批）进行人工把关的场景。
