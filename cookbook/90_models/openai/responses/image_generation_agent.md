# image_generation_agent.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example: Using the OpenAITools Toolkit for Image Generation

This script demonstrates how to use the `OpenAITools` toolkit, which includes a tool for generating images using OpenAI's DALL-E within an Agno Agent.

Example prompts to try:
- "Create a surreal painting of a floating city in the clouds at sunset"
- "Generate a photorealistic image of a cozy coffee shop interior"
- "Design a cute cartoon mascot for a tech startup"
- "Create an artistic portrait of a cyberpunk samurai"

Run `uv pip install openai agno` to install the necessary dependencies.
"""

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.openai import OpenAITools
from agno.utils.media import save_base64_data

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[OpenAITools(image_model="gpt-image-1")],
    markdown=True,
)

response = agent.run(
    "Generate a photorealistic image of a cozy coffee shop interior",
)

if response.images and response.images[0].content:
    save_base64_data(str(response.images[0].content), "tmp/coffee_shop.png")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/openai/responses/image_generation_agent.py`

## 概述

本示例展示 Agno 的 **`OpenAIChat` + `OpenAITools` 图像生成** 机制：通过 Chat Completions 路径调用 `OpenAITools`（含 DALL-E/`gpt-image-1`），并从 `RunOutput.images` 取 Base64 落盘。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | **Chat Completions**，非 Responses |
| `tools` | `[OpenAITools(image_model="gpt-image-1")]` | OpenAI 官方工具包（含生图） |
| `markdown` | `True` | Markdown 附加段 |

## 架构分层

```
用户代码层                agno.agent 层
┌──────────────────┐    ┌──────────────────────────────────┐
│ agent.run(...)   │───>│ 工具调用链 → 生图 → response.images   │
│ OpenAIChat       │    │ chat.completions.create              │
└──────────────────┘    └──────────────────────────────────┘
```

## 核心组件解析

### OpenAITools(image_model=...)

配置图像子模型；Agent 执行后若返回图像，保存在 `response.images`。

### 运行机制与因果链

1. **路径**：用户自然语言 → 模型决定调用生图工具 → 返回含图像内容的 `RunOutput`。
2. **状态**：`save_base64_data` 写本地 `tmp/coffee_shop.png`，无 DB。
3. **分支**：无工具则仅文本；本示例依赖工具分支产出图。
4. **定位**：同目录多数文件用 **Responses**，本文件刻意用 **Chat + OpenAITools** 演示生图。

## System Prompt 组装

无显式 `instructions`/`description`；`markdown=True`。

### 还原后的完整 System 文本

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>

```

## 完整 API 请求

```python
# Chat Completions（OpenAIChat.invoke ~L412）
client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    tools=[...],  # OpenAITools 展开 schema
)
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["用户描述画面"] --> B["【关键】OpenAITools 生图"]
    B --> C["RunOutput.images"]
    C --> D["save_base64_data"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/openai/chat.py` | `invoke()` L385 | Chat 调用 |
| `agno/tools/openai/` | `OpenAITools` | 生图工具 |
