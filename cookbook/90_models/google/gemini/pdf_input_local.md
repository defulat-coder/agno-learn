# pdf_input_local.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Google Pdf Input Local
======================

Cookbook example for `google/gemini/pdf_input_local.py`.
"""

from pathlib import Path

from agno.agent import Agent
from agno.media import File
from agno.models.google import Gemini
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
    model=Gemini(id="gemini-3-flash-preview"),
    markdown=True,
    add_history_to_context=True,
)

agent.print_response(
    "Summarize the contents of the attached file.",
    files=[File(filepath=pdf_path)],
)
agent.print_response("Suggest me a recipe from the attached file.")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/google/gemini/pdf_input_local.py`

## 概述

**本地下载 PDF**，`File(filepath=pdf_path)`，大文件可自动上传 Google（见文件头注释），两轮对话。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-3-flash-preview")` | |
| `add_history_to_context` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["File(filepath)"] --> B["【关键】本地 PDF 进 Gemini"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/utils/media.py` | `download_file` | 拉取 PDF |
