# image_agent.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Dashscope Image Agent
=====================

Cookbook example for `dashscope/image_agent.py`.
"""

import asyncio

from agno.agent import Agent
from agno.media import Image
from agno.models.dashscope import DashScope
from agno.tools.websearch import WebSearchTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=DashScope(id="qwen-vl-plus"),
    tools=[WebSearchTools()],
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    # --- Sync + Streaming ---
    agent.print_response(
        "Analyze this image in detail and tell me what you see. Also search for more information about the subject.",
        images=[
            Image(
                url="https://upload.wikimedia.org/wikipedia/commons/0/0c/GoldenGateBridge-001.jpg"
            )
        ],
        stream=True,
    )

    # --- Async + Streaming ---
    async def main():
        await agent.aprint_response(
            "What do you see in this image? Provide a detailed description and search for related information.",
            images=[
                Image(
                    url="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Cat03.jpg/1200px-Cat03.jpg"
                )
            ],
            stream=True,
        )

    asyncio.run(main())
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/dashscope/image_agent.py`

## 概述

本示例展示 **DashScope 视觉模型 `qwen-vl-plus` + WebSearchTools**：图像理解与联网搜索组合。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `DashScope(id="qwen-vl-plus")` | 多模态 + Chat Completions |
| `tools` | `[WebSearchTools()]` | 工具调用 |
| `markdown` | `True` | Markdown system 段 |

## 核心组件解析

### 运行机制与因果链

1. **路径**：user 含图像 URL + 文本 → 模型可能调用搜索工具 → 回答。
2. **副作用**：网络搜索；无 db。
3. **分支**：同步流式与 `async` 流式两条演示。

## System Prompt 组装

含工具指令与 Markdown；无自定义 `description`。

## 完整 API 请求

`chat.completions.create`，`messages` 含多模态 user，`tools` 为搜索工具定义。

## Mermaid 流程图

```mermaid
flowchart TD
    A["图像 URL + 用户问题"] --> B["【关键】qwen-vl-plus 多模态"]
    B --> C["工具循环 / WebSearch"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/dashscope/dashscope.py` | `DashScope` | Qwen 兼容 API |
| `agno/models/openai/chat.py` | `invoke()` | Completions |
