# structured_output.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Nebius Structured Output
========================

Cookbook example for `nebius/structured_output.py`.
"""

from typing import List

from agno.agent import Agent, RunOutput  # noqa
from agno.models.nebius import Nebius
from pydantic import BaseModel, Field
from rich.pretty import pprint  # noqa

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


class MovieScript(BaseModel):
    setting: str = Field(
        ..., description="Provide a nice setting for a blockbuster movie."
    )
    ending: str = Field(
        ...,
        description="Ending of the movie. If not available, provide a happy ending.",
    )
    genre: str = Field(
        ...,
        description="Genre of the movie. If not available, select action, thriller or romantic comedy.",
    )
    name: str = Field(..., description="Give a name to this movie")
    characters: List[str] = Field(..., description="Name of characters for this movie.")
    storyline: str = Field(
        ..., description="3 sentence storyline for the movie. Make it exciting!"
    )


# Agent that uses a structured output
structured_output_agent = Agent(
    model=Nebius(id="Qwen/Qwen3-30B-A3B"),
    description="You are a helpful assistant. Summarize the movie script based on the location in a JSON object.",
    output_schema=MovieScript,
)

structured_output_agent.print_response("New York")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/nebius/structured_output.py`

## 概述

本示例展示 **`Nebius` + `output_schema=MovieScript`** 与字面量 `description`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Nebius(id="Qwen/Qwen3-30B-A3B")` | `supports_json_schema_outputs` 继承 OpenAIChat |
| `description` | 见下 | 静态 |
| `output_schema` | `MovieScript` | Pydantic |

## System Prompt 组装

### 还原后的完整 System 文本（description 原样）

```text
You are a helpful assistant. Summarize the movie script based on the location in a JSON object.
```

用户消息：`"New York"`

## Mermaid 流程图

```mermaid
flowchart TD
    A["description + schema"] --> B["【关键】response_format json_schema"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/openai/chat.py` | `get_request_params` |
