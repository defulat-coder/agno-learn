# instantiate_team.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/instantiate_team.py`

## 概述

本示例测量 **`Team(members=[team_member])` 构造** 耗时，`num_iterations=1000`，无 `run`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `team_member` | `Agent(model=OpenAIChat gpt-4o)` | 单成员 |

## 核心组件解析

对比 `instantiate_agent`：Team 协调器与成员图初始化成本。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/team/team.py` | `Team.__init__` |
