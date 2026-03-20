# pdf_input_local.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Anthropic Pdf Input Local
=========================

Cookbook example for `anthropic/pdf_input_local.py`.
"""

from pathlib import Path

from agno.agent import Agent
from agno.media import File
from agno.models.anthropic import Claude
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
    model=Claude(id="claude-sonnet-4-20250514"),
    markdown=True,
)

agent.print_response(
    "Summarize the contents of the attached file.",
    files=[
        File(
            filepath=pdf_path,
        ),
    ],
)
run_response = agent.get_last_run_output()
print("Citations:")
print(run_response.citations)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/anthropic/pdf_input_local.py`

## 概述

本示例展示 **`File(filepath=...)`** 传入本地 PDF 路径，并打印 **citations**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Claude(id="claude-sonnet-4-20250514")` | 文档 |
| `markdown` | `True` | Markdown |
| `files` | `File(filepath=pdf_path)` | 本地 PDF |

## 运行机制与因果链

框架读取本地文件并编码；适合与 `pdf_input_bytes` / `pdf_input_url` 对比。

## System Prompt 组装

### 还原后的完整 System 文本

```text
Use markdown to format your answers.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["File(filepath)"] --> B["读本地 PDF"]
    B --> C["Claude"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/media` | `File` | 文件封装 |
