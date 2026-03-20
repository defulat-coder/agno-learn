# unsplash_tools.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Unsplash Tools Example

This example demonstrates how to use the UnsplashTools toolkit with an AI agent
to search for and retrieve high-quality, royalty-free images from Unsplash.

UnsplashTools provides:
- search_photos: Search photos by keyword with filters (orientation, color)
- get_photo: Get detailed info about a specific photo
- get_random_photo: Get random photo(s) with optional query
- download_photo: Track downloads for Unsplash API compliance

Setup:
1. Get a free API key from https://unsplash.com/developers
2. Set the environment variable: export UNSPLASH_ACCESS_KEY="your_access_key"
3. Install dependencies: pip install openai agno

Usage:
    python unsplash_tools.py
"""

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.unsplash import UnsplashTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


# Example 1: Basic usage with default tools
# By default, search_photos, get_photo, and get_random_photo are enabled
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[UnsplashTools()],
    instructions=[
        "You are a helpful assistant that can search for high-quality images.",
        "When presenting image results, include the image URL, author name, and description.",
        "Always credit the photographer by including their name and Unsplash profile link.",
    ],
    markdown=True,
)

# Example 2: Enable all tools including download tracking
# Use this when you need to comply with Unsplash's download tracking requirement
agent_with_download = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[UnsplashTools(enable_download_photo=True)],
    instructions=[
        "You are a helpful assistant that can search for high-quality images.",
        "When a user wants to use/download an image, use the download_photo tool to track it.",
    ],
    markdown=True,
)

# Example 3: Enable only specific tools
agent_search_only = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[
        UnsplashTools(
            enable_search_photos=True,
            enable_get_photo=False,
            enable_get_random_photo=False,
        )
    ],
    markdown=True,
)

# Run examples
# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    # Search for photos
    print("=" * 60)
    print("Example 1: Searching for nature photos")
    print("=" * 60)
    agent.print_response(
        "Find me 3 beautiful landscape photos of mountains",
        stream=True,
    )

    # Get a random photo
    print("\n" + "=" * 60)
    print("Example 2: Getting a random photo")
    print("=" * 60)
    agent.print_response(
        "Get me a random photo of a coffee shop",
        stream=True,
    )

    # Search with filters
    print("\n" + "=" * 60)
    print("Example 3: Search with orientation filter")
    print("=" * 60)
    agent.print_response(
        "Find 2 portrait-oriented photos of city skylines at night",
        stream=True,
    )

# --- Download Compliance Note ---
#
# The download_photo tool exists for Unsplash API compliance.
# According to Unsplash API guidelines, you must trigger the download endpoint
# when a photo is actually downloaded or used in your application.
#
# What download_photo does:
# - Calls /photos/{id}/download to increment the photographer's download count
# - Returns a time-limited download URL
# - Does NOT download the image file itself
#
# This is required for proper attribution tracking and is part of Unsplash's
# terms of service. The tool is disabled by default (enable_download_photo=False)
# since it's only needed when actually using/downloading images.
#
# See: https://unsplash.com/documentation#track-a-photo-download
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/unsplash_tools.py`

## 概述

Unsplash Tools Example

本示例归类：**单 Agent**；模型相关类型：`OpenAIChat`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | OpenAIChat(id='gpt-4o'…) | `Agent(...)` |
| `markdown` | True | `Agent(...)` |
| （Model 类） | `OpenAIChat` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ unsplash_tools.py    │  ──▶  │ Agent → get_run_messages → Model │
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
