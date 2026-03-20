# 2_tools.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Agent with Tools - Finance Research Agent
==========================================
Give an agent tools to search the web and take real-world actions.

Key concepts:
- tools: A list of Toolkit instances the agent can call
- instructions: System-level guidance that shapes the agent's behavior
- add_datetime_to_context: Injects the current date/time so the agent knows "today"
- WebSearchTools: Built-in toolkit for web search via DuckDuckGo (no API key needed)

Example prompts to try:
- "Compare the latest funding rounds in AI startups this month"
- "What's happening with interest rates this week?"
- "Find the latest news about Nvidia's earnings"
- "What are the top tech IPOs planned for this quarter?"
"""

from agno.agent import Agent
from agno.models.google import Gemini
from agno.tools.websearch import WebSearchTools

# ---------------------------------------------------------------------------
# Agent Instructions
# ---------------------------------------------------------------------------
instructions = """\
You are a finance research agent. You find and analyze current financial news.

## Workflow

1. Search the web for the requested financial information
2. Analyze and compare findings
3. Present a clear, structured summary

## Rules

- Always cite your sources
- Use tables for comparisons
- Include dates for all data points\
"""

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
finance_agent = Agent(
    name="Finance Agent",
    model=Gemini(id="gemini-3-flash-preview"),
    instructions=instructions,
    tools=[WebSearchTools()],
    # Adds current date/time to the system message so the agent knows "today"
    add_datetime_to_context=True,
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    finance_agent.print_response(
        "Compare the latest funding rounds in AI startups this month",
        stream=True,
    )

# ---------------------------------------------------------------------------
# More Examples
# ---------------------------------------------------------------------------
"""
Tools are Python classes that inherit from Toolkit. Agno includes many built-in:

1. Web search (no API key needed)
   from agno.tools.websearch import WebSearchTools
   tools=[WebSearchTools()]

2. Yahoo Finance (real market data)
   from agno.tools.yfinance import YFinanceTools
   tools=[YFinanceTools(all=True)]

3. Exa search (semantic search, needs EXA_API_KEY)
   from agno.tools.exa import ExaTools
   tools=[ExaTools()]

4. Custom tools
   @tool
   def my_tool(query: str) -> str:
       return "result"

You can combine multiple toolkits:
   tools=[WebSearchTools(), YFinanceTools(all=True)]

The agent decides which tool to call based on the prompt.
"""
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/gemini_3/2_tools.py`

## 概述

Agent with Tools - Finance Research Agent

本示例归类：**单 Agent**；模型相关类型：`Gemini`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | 'Finance Agent' | `Agent(...)` |
| `model` | Gemini(id='gemini-3-flash-preview'…) | `Agent(...)` |
| `instructions` | 'You are a finance research agent. You find and analyze current financial news.\n\n## Workflow\n\n1. Search the web for th...' | `Agent(...)` |
| `add_datetime_to_context` | True | `Agent(...)` |
| `markdown` | True | `Agent(...)` |
| （Model 类） | `Gemini` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ 2_tools.py           │  ──▶  │ Agent → get_run_messages → Model │
└──────────────────────┘         └────────────────────────────────┘
                                          │
                                          ▼
                                  ┌───────────────┐
                                  │ 对应 Model 子类 │
                                  └───────────────┘
```

## 核心组件解析

### 运行机制与因果链

1. **入口**：从模块 `__main__` 或暴露的 `agent` / `team` 调用进入；同步用 `print_response` / `run`，异步用 `aprint_response` / `arun`（若源码中有）。
2. **消息**：默认路径下 system 内容由 `get_system_message()`（`libs/agno/agno/agent/_messages.py` 约 **L106** 起）按分段逻辑拼装；若显式传入 `system_message` 则早退使用该字符串。
3. **模型**：具体 HTTP/SDK 形态以 `libs/agno/agno/models/` 下对应类的 `invoke` / `ainvoke` 为准（勿默认写成单一 `chat.completions`）。
4. **副作用**：若配置 `db`、`knowledge`、`memory`，运行会读写存储；仅以本文件为准对照。

### 与框架的衔接

- **System**：`get_system_message()` 锚点 `agno/agent/_messages.py` **L106+**。
- **运行**：`Agent.print_response` 等入口 `agno/agent/agent.py`（以当前仓库检索为准）。

## System Prompt 组装

| 序号 | 组成部分 | 本文件 | 是否生效 |
|------|---------|--------|---------|
| 1 | `instructions` / `description` 等 | 见核心配置表与源码 | 有赋值则生效 |
| 2 | 默认分段（markdown、时间等） | 取决于 `Agent` 默认与显式参数 | 视参数 |

### 拼装顺序与源码锚点

1. `system_message` 直给 → 使用该内容（见 `_messages.py` 文档字符串分支说明）。
2. 否则默认拼装：`description`、`role`、`instructions`、markdown 附加段等按 `# 3.x` 注释顺序合并。

### 还原后的完整 System 文本

```text
--- instructions ---
You are a finance research agent. You find and analyze current financial news.

## Workflow

1. Search the web for the requested financial information
2. Analyze and compare findings
3. Present a clear, structured summary

## Rules

- Always cite your sources
- Use tables for comparisons
- Include dates for all data points
```

### 段落释义（模型视角）

- 指令与安全边界由 `instructions` / `system_message` 约束；若带 `tools` / `knowledge`，文档中需体现「何时检索/调用」由框架注入的提示段支持。

## 完整 API 请求

```python
# 请以本文件实际 Model 为准打开 libs/agno/agno/models/<厂商>/ 下对应类的 invoke：
# 可能是 chat.completions.create、responses.create、Gemini generate_content 等。
```

> 与上一节 system 文本在同一 run 中组合；`developer`/`system` 角色由适配器转换。

```mermaid
flowchart TD
    Entry["用户入口<br/>`if __name__` / main"] --> Run["【关键】Agent.run / print_response"]
    Run --> Sys["【关键】get_system_message<br/>agno/agent/_messages.py L106+"]
    Sys --> Inv["【关键】Model.invoke / 提供商 API"]
    Inv --> Out["RunOutput / 流式 chunk"]
```

**【关键】节点说明：**

- **print_response / run**：用户可见的同步入口。
- **get_system_message**：系统提示拼装核心。
- **Model.invoke**：对模型提供商的实际请求。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/_messages.py` | `get_system_message()` L106+ |
| `agno/agent/agent.py` | `Agent` 运行与 CLI 输出 |
| `agno/models/` | 各厂商 `Model.invoke` |
