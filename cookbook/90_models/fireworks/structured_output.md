# structured_output.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Fireworks Structured Output
===========================

Cookbook example for `fireworks/structured_output.py`.
"""

from typing import List

from agno.agent import Agent, RunOutput  # noqa
from agno.models.fireworks import Fireworks
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


# Agent that uses JSON mode
agent = Agent(
    model=Fireworks(id="accounts/fireworks/models/llama-v3p1-405b-instruct"),
    description="You write movie scripts.",
    output_schema=MovieScript,
)

# Get the response in a variable
response: RunOutput = agent.run("New York")
pprint(response.content)

# agent.print_response("New York")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/fireworks/structured_output.py`

## 概述

**Fireworks + `output_schema`**，`description="You write movie scripts."`，`agent.run("New York")` + `pprint`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Fireworks(id="accounts/fireworks/models/llama-v3p1-405b-instruct")` | |
| `description` | `You write movie scripts.` | 字面量 |
| `output_schema` | `MovieScript` | |

## System Prompt 组装

### 还原后的完整 System 文本

```text
You write movie scripts.

（动态 JSON/schema 提示段）
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["MovieScript schema"] --> B["【关键】结构化输出解析"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/_messages.py` | `get_system_message` | schema 段 |
