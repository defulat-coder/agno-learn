# external_url_input.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Example: Analyze files from public HTTPS URLs.

The Gemini API now supports external HTTPS URLs (up to 100MB).
Pass public URLs directly without downloading first.

This works with:
- Public URLs (no authentication required)
- Pre-signed URLs from AWS S3
- SAS URLs from Azure Blob Storage
- Any accessible HTTPS URL

Supported formats: PDF, JSON, HTML, CSS, XML, images (PNG, JPEG, WebP, GIF)

Note: External URL support requires Gemini 3.x models (e.g., gemini-3-flash-preview).
      Gemini 2.0 models do not support this feature.
"""

from agno.agent import Agent
from agno.media import File
from agno.models.google import Gemini

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=Gemini(id="gemini-3-flash-preview"),
    markdown=True,
)

# Pass public URL directly - Gemini fetches the content
agent.print_response(
    "Summarize this document.",
    files=[
        File(
            url="https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf",
            mime_type="application/pdf",
        )
    ],
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/google/gemini/external_url_input.py`

## 概述

**公网 HTTPS URL 直传** PDF（无需本地下载），需 **Gemini 3.x**；`File(url=..., mime_type="application/pdf")`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-3-flash-preview")` | |
| `markdown` | `True` | |

## 运行机制与因果链

Gemini 服务端拉取 URL 内容；区别于本地上传与 GCS。

## 完整 API 请求

`generate_content`，contents 含文件 URL 引用。

## Mermaid 流程图

```mermaid
flowchart TD
    A["File(url=https...)"] --> B["【关键】外部 HTTPS 文档"]
    B --> C["generate_content"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `_format_messages` | URL 文件 |
