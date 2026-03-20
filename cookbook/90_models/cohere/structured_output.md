# structured_output.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Cohere Structured Output
========================

Cookbook example for `cohere/structured_output.py`.
"""

import asyncio
from typing import List

from agno.agent import Agent, RunOutput  # noqa
from agno.models.cohere import Cohere
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


structured_output_agent = Agent(
    model=Cohere(id="command-a-03-2025"),
    description="You help people write movie scripts.",
    output_schema=MovieScript,
)

# Get the response in a variable
response: RunOutput = structured_output_agent.run("New York")
pprint(response.content)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    # --- Async ---
    asyncio.run(
        structured_output_agent.aprint_response(
            "Find a cool movie idea about London and write it."
        )
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/cohere/structured_output.py`

## 概述

**Cohere + MovieScript**，同步 `run` + `pprint`，并 **async `aprint_response`** 第二段提示。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cohere(id="command-a-03-2025")` | Cohere |
| `description` | `"You help people write movie scripts."` | system |
| `output_schema` | `MovieScript` | 结构化 |

## System Prompt 组装

### 还原后的完整 System 文本（核心）

```text
You help people write movie scripts.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["output_schema"] --> B["【关键】chat + response_format"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/cohere/chat.py` | `get_request_params` | 结构化参数 |
