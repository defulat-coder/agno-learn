# pdf_input_file_upload.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
In this example, we upload a PDF file to Anthropic directly and then use it as an input to an agent.
"""

from pathlib import Path

from agno.agent import Agent
from agno.media import File
from agno.models.anthropic import Claude
from agno.utils.media import download_file
from anthropic import Anthropic

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

pdf_path = Path(__file__).parent.joinpath("ThaiRecipes.pdf")

# Download the file using the download_file function
download_file(
    "https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf", str(pdf_path)
)

# Initialize Anthropic client
client = Anthropic()

# Upload the file to Anthropic
uploaded_file = client.beta.files.upload(
    file=Path(pdf_path),
)

if uploaded_file is not None:
    agent = Agent(
        model=Claude(id="claude-opus-4-20250514", betas=["files-api-2025-04-14"]),
        markdown=True,
    )

    agent.print_response(
        "Summarize the contents of the attached file.",
        files=[File(external=uploaded_file)],
    )

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/anthropic/pdf_input_file_upload.py`

## 概述

本示例展示 **Files API 上传 PDF** 后使用 **`File(external=uploaded_file)`** 与 `claude-opus` + files beta。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Claude(id="claude-opus-4-20250514", betas=["files-api-2025-04-14"])` | Files beta |
| `markdown` | `True` | Markdown |
| `files` | `File(external=uploaded_file)` | 引用上传结果 |

## 运行机制与因果链

1. **路径**：`beta.files.upload` → file 句柄 → `beta.messages` 引用文档。
2. **副作用**：云端保留上传文件。
3. **定位**：大文件优先走 **托管上传** 而非内联 bytes。

## System Prompt 组装

### 还原后的完整 System 文本

```text
Use markdown to format your answers.
```

## Mermaid 流程图

```mermaid
flowchart TD
    U["files.upload PDF"] --> F["File(external=...)"]
    F --> M["【关键】beta.messages"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/anthropic/claude.py` | `invoke` / beta | API 路径 |
