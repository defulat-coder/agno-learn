# 04_team_tool_metrics.py — 实现原理分析

> 源文件：`cookbook/03_teams/22_metrics/04_team_tool_metrics.py`

## 概述

本示例展示 **工具调用级别的指标**：除 Team 汇总指标和成员 Agent 指标外，还可以深入到每个工具调用的 `tool_call.metrics`（执行耗时、状态等），适合分析工具调用的性能瓶颈。

**核心配置一览：**

| 指标层级 | 获取方式 |
|---------|---------|
| 运行级汇总 | `run_output.metrics` |
| 成员级 | `member_response.metrics` |
| 工具调用级 | `tool_call.metrics` |

## 核心组件解析

### 工具调用指标读取

```python
for member_response in run_output.member_responses:
    print(f"Member: {member_response.agent_name}")
    pprint(member_response.metrics)  # 成员级
    
    if member_response.tools:
        for tool_call in member_response.tools:
            print(f"Tool: {tool_call.tool_name}")
            if tool_call.metrics:
                pprint(tool_call.metrics)  # 工具调用级（耗时等）
```

### `store_member_responses=True` 的必要性

无此配置时 `run_output.member_responses` 为空，工具级指标无法获取。

### 工具指标内容

`tool_call.metrics` 通常包含：
- 工具执行耗时（秒）
- 调用时间戳
- 成功/失败状态

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/team/team.py` | `store_member_responses` | 持久化成员响应 |
| `agno/run/base.py` | `ToolCallMetrics` | 工具调用指标结构 |
