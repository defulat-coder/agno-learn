# pdf_input_url.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Openai Pdf Input Url
====================

Cookbook example for `openai/chat/pdf_input_url.py`.
"""

from agno.agent import Agent
from agno.media import File
from agno.models.openai import OpenAIChat

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    markdown=True,
    add_history_to_context=True,
)

agent.print_response(
    "Suggest me a recipe from the attached file.",
    files=[File(url="https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf")],
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/openai/chat/pdf_input_url.py`

## 概述

**`File(url=...)`** 直接传 PDF URL，无需本地先下载。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | Chat |
| `markdown` | `True` | 默认 |
| `add_history_to_context` | `True` | 历史 |

用户消息：`"Suggest me a recipe from the attached file."`

## Mermaid 流程图

```mermaid
flowchart TD
    A["File(url)"] --> B["【关键】远程 PDF 取回"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/media` | `File` |
