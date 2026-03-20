# x402scan_mcp_tools.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
x402scan MCP Tools
==================
Give your agent money. Agents pay for APIs autonomously using USDC on Base.
Access 100+ paid data sources: enrichment, scraping, maps, social media, media generation.

Installation: npx @x402scan/mcp install
Documentation: https://x402scan.com/mcp

First run auto-generates a wallet at ~/.x402scan-mcp/wallet.json.
Fund with USDC on Base to start using paid APIs.
"""

import asyncio

from agno.agent import Agent
from agno.models.anthropic import Claude
from agno.team import Team
from agno.tools.mcp import MCPTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


async def run_agent(message: str) -> None:
    async with MCPTools("npx -y @x402scan/mcp@latest") as x402:
        agent = Agent(
            model=Claude(id="claude-sonnet-4-20250514"),
            tools=[x402],
            markdown=True,
        )
        await agent.aprint_response(message, stream=True)


async def run_team(message: str) -> None:
    async with MCPTools("npx -y @x402scan/mcp@latest") as x402:
        researcher = Agent(
            model=Claude(id="claude-sonnet-4-20250514"),
            tools=[x402],
            name="Researcher",
            role="Data Researcher",
            instructions=(
                "Gather data from paid APIs.\n"
                "Check balance before spending.\n"
                "Stay under $2 per task."
            ),
        )
        analyst = Agent(
            model=Claude(id="claude-sonnet-4-20250514"),
            name="Analyst",
            role="Data Analyst",
            instructions="Analyze data from the researcher. Create summaries and recommendations.",
        )
        team = Team(
            members=[researcher, analyst],
            instructions="Researcher gathers paid data, Analyst synthesizes findings.",
        )
        await team.aprint_response(message, stream=True)


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    # Onboarding: wallet setup and API discovery
    asyncio.run(run_agent("Show me my wallet info and available APIs"))

    # People enrichment (Apollo)
    # asyncio.run(run_agent("Find information about the CEO of Anthropic"))

    # Web scraping (Firecrawl)
    # asyncio.run(run_agent("Scrape the content of https://docs.agno.com/introduction"))

    # Search (Exa)
    # asyncio.run(run_agent("Search for recent AI agent framework comparisons"))

    # Google Maps
    # asyncio.run(run_agent("Find coffee shops near Times Square, New York"))

    # Social media (Grok/Twitter)
    # asyncio.run(run_agent("Search Twitter for recent posts about AI agents"))

    # Multi-agent team: researcher + analyst sharing a wallet
    # asyncio.run(run_team("Research VC funding trends in AI agents over the past 6 months"))
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/x402scan_mcp_tools.py`

## 概述

x402scan MCP Tools

本示例归类：**Team 多智能体**；模型相关类型：`Claude`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | Claude(id='claude-sonnet-4-20250514'…) | `Agent(...)` |
| `markdown` | True | `Agent(...)` |
| （Model 类） | `Claude` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ x402scan_mcp_tools.py │  ──▶  │ Agent → get_run_messages → Model │
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
（主 `Agent(...)` 未传入可静态解析的 `description`/`instructions`/`system_message` 字符串；此时 system 由 `get_system_message()` 默认段与 `markdown` 等开关决定，请在 `agno/agent/_messages.py` 对照分段注释，或在运行中打印 `get_system_message` 返回值。）
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
