# retries.py — 实现原理分析

> 源文件：`cookbook/02_agents/14_advanced/retries.py`

## 概述

本示例展示 Agno Agent 的**自动重试**机制：通过 `retries`、`delay_between_retries`、`exponential_backoff` 三个参数配置，当 Agent run 遇到错误时自动重试，支持指数退避策略。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Web Search Agent"` | Agent 名称 |
| `role` | `"Search the web for information"` | Agent 角色 |
| `tools` | `[WebSearchTools()]` | Web 搜索工具 |
| `retries` | `3` | 失败后最多重试 3 次 |
| `delay_between_retries` | `1` | 重试间隔（秒） |
| `exponential_backoff` | `True` | 每次重试延迟翻倍 |

## 重试机制说明

```
第1次尝试失败 → 等待 1s → 第2次尝试失败 → 等待 2s → 第3次尝试失败 → 等待 4s → 第4次尝试
                                                              ↑ 指数退避: 1, 2, 4...
```

| 参数 | 作用 |
|------|------|
| `retries=3` | 最多额外重试 3 次（共 4 次机会） |
| `delay_between_retries=1` | 基础等待秒数 |
| `exponential_backoff=True` | 延迟 = delay × 2^(attempt-1) |

## 核心代码模式

```python
agent = Agent(
    name="Web Search Agent",
    role="Search the web for information",
    tools=[WebSearchTools()],
    retries=3,
    delay_between_retries=1,
    exponential_backoff=True,
)

agent.print_response("What exactly is an AI Agent?", stream=True)
```

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.2.4 | `add_name_to_context` | "Web Search Agent" | 是 |
| 3.3.2 | `role` | "Search the web for information" | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `Agent(retries=, delay_between_retries=, exponential_backoff=)` | 重试配置入口 |
| `agno/agent/agent.py` | `run()` 内重试逻辑 | 捕获异常后按配置重试 |
