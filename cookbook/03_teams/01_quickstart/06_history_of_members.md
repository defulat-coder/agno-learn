# 06_history_of_members.py — 实现原理分析

> 源文件：`cookbook/03_teams/01_quickstart/06_history_of_members.py`

## 概述

本示例展示 Agno 的 **成员级历史（add_history_to_context on Agent）** 与 **TeamMode.route**：`german_agent` / `spanish_agent` 均 `add_history_to_context=True`，各自 DB 中保留**自己**的多轮上下文；队长 `mode=TeamMode.route`；`determine_input_for_members=False`。

**核心配置一览：**

| 配置项 | 值 |
|--------|-----|
| `mode` | `TeamMode.route` |
| 成员 `add_history_to_context` | `True` |
| `determine_input_for_members` | `False` |
| `db` | `SqliteDb(...)` |

## 核心组件解析

与 `05_team_history.py`（团队历史到成员）对照：本例强调 **Agent 私有历史**，同一 session 下德语线程与西语线程分别延续。

## System Prompt 组装

队长 instructions 同系列多语言委派（与 05 类似两行）。

### 还原后的完整 System 文本

```text
You are a multi lingual Q and A team that can answer questions in English and Spanish. You MUST delegate the task to the appropriate member based on the language of the question.
If the question is in German, delegate to the German agent. If the question is in Spanish, delegate to the Spanish agent.
```

## 完整 API 请求

`OpenAIResponses(id="gpt-5.2")`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["成员 Agent"] --> B["【关键】add_history_to_context"]
    B --> C["成员私有会话链"]
```

- **【关键】add_history_to_context**：成员侧历史，区别于团队级 `add_team_history_to_members`。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/agent.py` | 成员 `add_history_to_context` |
| `agno/team/team.py` | `mode` |
