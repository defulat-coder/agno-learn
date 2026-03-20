# image_bytes_input_agent.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Mistral Image Bytes Input Agent
===============================

Cookbook example for `mistral/image_bytes_input_agent.py`.
"""

import requests
from agno.agent import Agent
from agno.media import Image
from agno.models.mistral.mistral import MistralChat

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=MistralChat(id="pixtral-12b-2409"),
    markdown=True,
)

image_url = (
    "https://tripfixers.com/wp-content/uploads/2019/11/eiffel-tower-with-snow.jpeg"
)


def fetch_image_bytes(url: str) -> bytes:
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
        "Accept": "image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8",
        "Accept-Language": "en-US,en;q=0.9",
    }
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.content


image_bytes_from_url = fetch_image_bytes(image_url)

agent.print_response(
    "Tell me about this image.",
    images=[
        Image(content=image_bytes_from_url),
    ],
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/mistral/image_bytes_input_agent.py`

## 概述

本示例展示 Agno 的 **多模态消息（`Image(content=bytes)`）+ Pixtral 视觉模型** 机制：用 `requests` 拉取图片字节，经 `MistralChat` 发往 Mistral 多模态 Chat API。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `MistralChat(id="pixtral-12b-2409")` | 视觉模型 |
| `markdown` | `True` | 默认 Markdown 提示 |

## 架构分层

用户层提供 `images=[Image(content=...)]` → `get_run_messages` 将图像编码进用户消息 → `MistralChat.invoke` → `format_messages`。

## 核心组件解析

### Image 与字节

`Image(content=image_bytes_from_url)` 由媒体层转为模型可消费的附件格式（见 `format_messages` / Mistral 侧约定）。

### 运行机制与因果链

1. **路径**：下载 JPEG → 作为用户消息图像部分 → 模型描述图像。
2. **副作用**：网络请求下载图片；无持久化。
3. **与 basic 差异**：用户消息含 **图像模态**。

## System Prompt 组装

无自定义 description/instructions；含 Markdown 默认句（若拼装路径启用）。

### 还原后的完整 System 文本

```text
Use markdown to format your answers.
```

（另含模型侧 instructions 时以运行时为准。）

用户消息：`"Tell me about this image."` + 图像附件。

## 完整 API 请求

Mistral `chat.complete`，`messages` 中含多模态 user 内容（文本 + 图像）。

## Mermaid 流程图

```mermaid
flowchart TD
    A["Image(bytes)"] --> B["get_run_messages"]
    B --> C["【关键】format_messages 多模态"]
    C --> D["MistralChat.invoke"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/utils/models/mistral.py` | `format_messages` | 消息格式 |
| `agno/models/mistral/mistral.py` | `MistralChat.invoke` | API 调用 |
