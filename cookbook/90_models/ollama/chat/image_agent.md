# image_agent.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Ollama Image Agent
==================

Cookbook example for `ollama/chat/image_agent.py`.
"""

from pathlib import Path

from agno.agent import Agent
from agno.media import Image
from agno.models.ollama import Ollama

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=Ollama(id="llama3.2-vision"),
    markdown=True,
)

image_path = Path(__file__).parent.joinpath("super-agents.png")
agent.print_response(
    "Write a 3 sentence fiction story about the image",
    images=[Image(filepath=image_path)],
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/ollama/chat/image_agent.py`

## 概述

**`Ollama(id="llama3.2-vision")` + 本地图** 图像理解。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Ollama(id="llama3.2-vision")` | 视觉 |
| `markdown` | `True` | 默认 |

用户消息：`"Write a 3 sentence fiction story about the image"` + `super-agents.png`

## Mermaid 流程图

```mermaid
flowchart TD
    A["llama3.2-vision"] --> B["【关键】vision 消息"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/ollama/chat.py` | 多模态 formatting |
