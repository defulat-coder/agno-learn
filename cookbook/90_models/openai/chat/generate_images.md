# generate_images.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Openai Generate Images
======================

Cookbook example for `openai/chat/generate_images.py`.
"""

from agno.agent import Agent, RunOutput
from agno.models.openai import OpenAIChat
from agno.tools.dalle import DalleTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

image_agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[DalleTools()],
    description="You are an AI agent that can generate images using DALL-E.",
    instructions="When the user asks you to create an image, use the `create_image` tool to create the image.",
    markdown=True,
)

image_agent.print_response("Generate an image of a white siamese cat")

# Retrieve and display generated images using get_last_run_output
run_response = image_agent.get_last_run_output()
if run_response and isinstance(run_response, RunOutput) and run_response.images:
    for image_response in run_response.images:
        image_url = image_response.url
        print(image_url)
else:
    print("No images found in run response")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/openai/chat/generate_images.py`

## 概述

**DalleTools + description + instructions**：gpt-4o 调用 `create_image` 生图，并从 `get_last_run_output().images` 取 URL。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | Chat |
| `tools` | `[DalleTools()]` | 文生图 |
| `description` | `"You are an AI agent that can generate images using DALL-E."` | 字面量 |
| `instructions` | `"When the user asks you to create an image, use the \`create_image\` tool..."` | 字面量 |
| `markdown` | `True` | 默认 |

## System Prompt 组装

两段用户字面量均进入默认 system 拼装（description → instructions）。

用户消息：`"Generate an image of a white siamese cat"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["DalleTools"] --> B["【关键】工具调用生成图像 URL"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/dalle.py` | `DalleTools` |
