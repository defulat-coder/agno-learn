# ai_news.py — 实现原理分析

> 源文件：`cookbook/09_evals/reliability/team/ai_news.py`

## 概述

本示例对 **`Team.run` 的 `TeamRunOutput`** 做可靠性检查：期望出现 **`delegate_task_to_member`** 与 **`search_news`**（`WebSearchTools(enable_news=True)`）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `ReliabilityEval.team_response` | `TeamRunOutput` | 非单 Agent |
| `expected_tool_calls` | 见源码列表 | Team 编排 + 新闻搜索 |

## 核心组件解析

验证 Team 级工具追踪字段与单 Agent `RunOutput` 的差异处理（见 `agno/eval/reliability.py`）。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/run/team.py` | `TeamRunOutput` |
| `agno/eval/reliability.py` | `team_response` |
