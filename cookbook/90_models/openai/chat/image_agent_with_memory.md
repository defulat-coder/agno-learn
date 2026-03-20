# image_agent_with_memory.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Openai Image Agent With Memory
==============================

Cookbook example for `openai/chat/image_agent_with_memory.py`.
"""

from agno.agent import Agent
from agno.media import Image
from agno.models.openai import OpenAIChat
from agno.tools.websearch import WebSearchTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[WebSearchTools()],
    markdown=True,
    add_history_to_context=True,
    num_history_runs=3,
)

agent.print_response(
    "Tell me about this image and give me the latest news about it.",
    images=[
        Image(
            url="https://upload.wikimedia.org/wikipedia/commons/0/0c/GoldenGateBridge-001.jpg"
        )
    ],
)

agent.print_response("Tell me where I can get more images?")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/openai/chat/image_agent_with_memory.py`

## 概述

**图像 + 工具 + `add_history_to_context` + `num_history_runs=3`**：第二轮仅文本追问图片来源。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | 视觉 |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `add_history_to_context` | `True` | 历史 |
| `num_history_runs` | `3` | 历史深度 |
| `markdown` | `True` | 默认 |

用户消息1：看图+新闻；用户消息2：`"Tell me where I can get more images?"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["首轮多模态"] --> B["【关键】次轮依赖消息历史"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/_messages.py` | 历史注入 |
