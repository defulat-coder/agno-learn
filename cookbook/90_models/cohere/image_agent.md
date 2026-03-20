# image_agent.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Cohere Image Agent
==================

Cookbook example for `cohere/image_agent.py`.
"""

from agno.agent import Agent
from agno.media import Image
from agno.models.cohere import Cohere

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=Cohere(id="c4ai-aya-vision-8b"),
    markdown=True,
)

agent.print_response(
    "Tell me about this image.",
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

> 源文件：`cookbook/90_models/cohere/image_agent.py`

## 概述

**Cohere 视觉模型 `c4ai-aya-vision-8b`** + **Image URL**，流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cohere(id="c4ai-aya-vision-8b")` | Vision |
| `markdown` | `True` | Markdown |
| `images` | `Image(url=...)` | 远程图 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["Vision"] --> B["【关键】chat + 图像消息"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/cohere/chat.py` | `format_messages` | 多模态 |
