# image_input.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Google Image Input
==================

Cookbook example for `google/gemini/image_input.py`.
"""

from agno.agent import Agent
from agno.media import Image
from agno.models.google import Gemini
from agno.tools.websearch import WebSearchTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=Gemini(id="gemini-2.0-flash-exp"),
    tools=[WebSearchTools()],
    markdown=True,
)

agent.print_response(
    "Tell me about this image and give me the latest news about it.",
    images=[
        Image(
            url="https://upload.wikimedia.org/wikipedia/commons/b/bf/Krakow_-_Kosciol_Mariacki.jpg"
        ),
    ],
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/google/gemini/image_input.py`

## 概述

**图像 URL + WebSearch**：`gemini-2.0-flash-exp`，先识图再联网。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-2.0-flash-exp")` | |
| `tools` | `[WebSearchTools()]` | |
| `markdown` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["Image(url)"] --> B["【关键】视觉 + 搜索工具"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `invoke` | generate_content |
