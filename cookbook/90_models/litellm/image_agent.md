# image_agent.md — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Litellm Image Agent
===================

Cookbook example for `litellm/image_agent.py`.
"""

from agno.agent import Agent
from agno.media import Image
from agno.models.litellm import LiteLLM
from agno.tools.websearch import WebSearchTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=LiteLLM(id="gpt-4o"),
    tools=[WebSearchTools()],
    markdown=True,
)

agent.print_response(
    "Tell me about this image and give me the latest news about it.",
    images=[
        Image(
            url="https://upload.wikimedia.org/wikipedia/commons/0/0c/GoldenGateBridge-001.jpg"
        )
    ],
    stream=True,
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/litellm/image_agent.py`

## 概述

**`LiteLLM(gpt-4o)` + WebSearch + 图像 URL**，流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LiteLLM(id="gpt-4o")` | 多模态 |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | Markdown |

## System Prompt 组装

用户消息：`Tell me about this image and give me the latest news about it.`，`images=[Image(url=...)]`。

## 完整 API 请求

`LiteLLM._format_messages` 将图像并入 messages；`completion` 发往支持视觉的模型。

## Mermaid 流程图

```mermaid
flowchart TD
    A["Image URL + WebSearch"] --> B["【关键】多模态 completion"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/litellm/chat.py` | `_format_messages` |
