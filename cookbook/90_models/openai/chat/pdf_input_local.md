# pdf_input_local.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Openai Pdf Input Local
======================

Cookbook example for `openai/chat/pdf_input_local.py`.
"""

from pathlib import Path

from agno.agent import Agent
from agno.media import File
from agno.models.openai import OpenAIChat
from agno.utils.media import download_file

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

pdf_path = Path(__file__).parent.joinpath("ThaiRecipes.pdf")

# Download the file using the download_file function
download_file(
    "https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf", str(pdf_path)
)

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    markdown=True,
    add_history_to_context=True,
)

agent.print_response(
    "What is the recipe for Gaeng Som Phak Ruam? Also what are the health benefits. Refer to the attached file.",
    files=[File(filepath=pdf_path)],
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/openai/chat/pdf_input_local.py`

## 概述

**下载 ThaiRecipes.pdf + `File(filepath=pdf_path)` + 长问答**（含健康益处）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | Chat |
| `markdown` | `True` | 默认 |
| `add_history_to_context` | `True` | 历史 |

用户消息：询问 Gaeng Som Phak Ruam 配方与健康益处，引用附件。

## Mermaid 流程图

```mermaid
flowchart TD
    A["本地 PDF"] --> B["【关键】长上下文文件理解"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/utils/media.py` | `download_file` |
