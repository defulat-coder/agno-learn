# langfuse_via_openinference_team.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Langfuse Team Tracing Via OpenInference
=======================================

Demonstrates sync and async team tracing with Langfuse.
"""

import asyncio
import base64
import os
from uuid import uuid4

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.team import Team
from agno.tools.websearch import WebSearchTools
from agno.tools.yfinance import YFinanceTools
from openinference.instrumentation.agno import AgnoInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import SimpleSpanProcessor

# ---------------------------------------------------------------------------
# Setup
# ---------------------------------------------------------------------------
LANGFUSE_AUTH = base64.b64encode(
    f"{os.getenv('LANGFUSE_PUBLIC_KEY')}:{os.getenv('LANGFUSE_SECRET_KEY')}".encode()
).decode()
os.environ["OTEL_EXPORTER_OTLP_ENDPOINT"] = (
    "https://us.cloud.langfuse.com/api/public/otel"  # US data region
)
# os.environ["OTEL_EXPORTER_OTLP_ENDPOINT"] = "https://cloud.langfuse.com/api/public/otel"  # EU data region
# os.environ["OTEL_EXPORTER_OTLP_ENDPOINT"] = "http://localhost:3000/api/public/otel"  # Local deployment (>= v3.22.0)

os.environ["OTEL_EXPORTER_OTLP_HEADERS"] = f"Authorization=Basic {LANGFUSE_AUTH}"

tracer_provider = TracerProvider()
tracer_provider.add_span_processor(SimpleSpanProcessor(OTLPSpanExporter()))

# Start instrumenting agno
AgnoInstrumentor().instrument(tracer_provider=tracer_provider)


# ---------------------------------------------------------------------------
# Create Team
# ---------------------------------------------------------------------------
# First agent for market data
market_data_agent = Agent(
    name="Market Data Agent",
    role="Fetch and analyze stock market data",
    id="market-data",
    model=OpenAIChat(id="gpt-4.1"),
    tools=[YFinanceTools()],
    instructions=[
        "You are a market data specialist.",
        "Focus on current stock prices and key metrics.",
        "Always present data in tables.",
    ],
)

# Second agent for news and research
news_agent = Agent(
    name="News Research Agent",
    role="Research company news",
    id="news-research",
    model=OpenAIChat(id="gpt-4.1"),
    tools=[WebSearchTools()],
    instructions=[
        "You are a financial news analyst.",
        "Focus on recent company news and developments.",
        "Always cite your sources.",
    ],
)

# Create team with both agents
financial_team = Team(
    name="Financial Analysis Team",
    id=str(uuid4()),
    user_id=str(uuid4()),
    model=OpenAIChat(id="gpt-4.1"),
    members=[
        market_data_agent,
        news_agent,
    ],
    instructions=[
        "Coordinate between market data and news analysis.",
        "First get market data, then relevant news.",
        "Combine the information into a clear summary.",
    ],
    show_members_responses=True,
    markdown=True,
)


# ---------------------------------------------------------------------------
# Run Team
# ---------------------------------------------------------------------------
def run_sync_example() -> None:
    financial_team.print_response(
        "Analyze Tesla (TSLA) stock - provide both current market data and recent significant news.",
        stream=True,
    )


async def run_async_example() -> None:
    await financial_team.aprint_response(
        "Analyze Tesla (TSLA) stock - provide both current market data and recent significant news.",
        stream=True,
    )


if __name__ == "__main__":
    run_mode = "sync"

    if run_mode == "async":
        asyncio.run(run_async_example())
    else:
        run_sync_example()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/92_integrations/observability/teams/langfuse_via_openinference_team.py`

## 概述

Langfuse Team Tracing Via OpenInference

本示例归类：**Team 多智能体**；模型相关类型：`OpenAIChat`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | 'Market Data Agent' | `Agent(...)` |
| `role` | 'Fetch and analyze stock market data' | `Agent(...)` |
| `id` | 'market-data' | `Agent(...)` |
| `model` | OpenAIChat(id='gpt-4.1'…) | `Agent(...)` |
| （Model 类） | `OpenAIChat` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ langfuse_via_openinference_team.py │  ──▶  │ Agent → get_run_messages → Model │
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
--- role ---
Fetch and analyze stock market data
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
