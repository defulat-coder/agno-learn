# hotel_management_typesafe.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Hotel Management Typesafe
=============================

Demonstrates hotel management typesafe.
"""

import asyncio
from datetime import date
from textwrap import dedent
from typing import List, Literal

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.mcp_toolbox import MCPToolbox
from pydantic import BaseModel, Field

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


url = "http://127.0.0.1:5001"

toolsets = ["hotel-management", "booking-system"]


class Hotel(BaseModel):
    id: int = Field(..., description="Unique identifier for the hotel")
    name: str = Field(..., description="Name of the hotel")
    location: str = Field(..., description="Location of the hotel")
    checkin_date: date = Field(..., description="Check-in date for the hotel stay")
    checkout_date: date = Field(..., description="Check-out date for the hotel stay")
    price_tier: Literal["Luxury", "Economy", "Boutique", "Extended-Stay"] = Field(
        description="The hotel tier/category - must be one of: Luxury, Economy, Boutique, or Extended-Stay"
    )
    booked: str = Field(
        description="Indicates if the hotel is booked (bit field from database)"
    )


class HotelSearch(BaseModel):
    location: str = Field(
        ...,
        description="The city, region, or specific location to search for hotels",
        min_length=1,
        max_length=100,
    )
    tier: Literal["Luxury", "Economy", "Boutique", "Extended-Stay"] = Field(
        description="The hotel tier/category to search for"
    )


class HotelSearchResult(BaseModel):
    hotels: List[Hotel] = Field(
        description="List of hotels matching the search criteria"
    )
    total_results: int = Field(description="Total number of hotels found")


agent = Agent(
    tools=[],
    instructions=dedent(
        """ \
        You're a helpful hotel assistant. You handle hotel searching, booking and
        cancellations. When the user searches for a hotel, mention it's name, id,
        location and price tier. Always mention hotel ids while performing any
        searches. This is very important for any operations. For any bookings or
        cancellations, please provide the appropriate confirmation. Be sure to
        update checkin or checkout dates if mentioned by the user.
        Don't ask for confirmations from the user.
    """
    ),
    markdown=True,
    input_schema=HotelSearch,
    output_schema=HotelSearchResult,
    parser_model=OpenAIChat("gpt-5.2"),
    debug_mode=True,
    debug_level=2,
)


async def run_agent(hotel_search: HotelSearch) -> None:
    async with MCPToolbox(url=url, toolsets=toolsets) as tools:
        agent.tools = [tools]
        await agent.aprint_response(hotel_search)


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    hotel_search = HotelSearch(
        location="Zurich",
        tier="Boutique",
    )

    asyncio.run(run_agent(hotel_search))
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/mcp/mcp_toolbox_demo/hotel_management_typesafe.py`

## 概述

Hotel Management Typesafe

本示例归类：**单 Agent**；模型相关类型：`OpenAIChat`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `instructions` | dedent("         You're a helpful hotel assistant. You handle hotel searching, booking and\n        cancellations. When the user searches for a hotel, mention it's name, id,\n        location and price tier. Always mention hotel ids while performing any\n        searches. This is very important for any operations. For any bookings or\n        cancellations, please provide the appropriate confirmation. Be sure to\n        update checkin or checkout dates if mentioned by the user.\n        Don't ask for confirmations from the user.\n    "…) | `Agent(...)` |
| `markdown` | True | `Agent(...)` |
| `input_schema` | 变量 `HotelSearch` | `Agent(...)` |
| `output_schema` | 变量 `HotelSearchResult` | `Agent(...)` |
| `parser_model` | OpenAIChat('gpt-5.2'…) | `Agent(...)` |
| `debug_mode` | True | `Agent(...)` |
| `debug_level` | 2 | `Agent(...)` |
| （Model 类） | `OpenAIChat` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ hotel_management_typesafe.py │  ──▶  │ Agent → get_run_messages → Model │
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
