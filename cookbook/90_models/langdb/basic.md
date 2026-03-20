# basic.md — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Langdb Basic
============

Cookbook example for `langdb/basic.py`.
"""

from agno.agent import Agent, RunOutput  # noqa
from agno.models.langdb import LangDB

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(model=LangDB(id="llama3-1-70b-instruct-v1.0"), markdown=True)

# Get the response in a variable
# run: RunOutput = agent.run("Share a 2 sentence horror story")
# print(run.content)

# Print the response in the terminal

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    # --- Sync ---
    agent.print_response("Share a 2 sentence horror story")

    # --- Sync + Streaming ---
    agent.print_response("Share a 2 sentence horror story", stream=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/langdb/basic.py`

## 概述

**`LangDB` 最小示例**，`llama3-1-70b-instruct-v1.0`，同步与流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LangDB(id="llama3-1-70b-instruct-v1.0")` | LangDB |
| `markdown` | `True` | Markdown |

## System Prompt 组装

静态附加：

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

用户消息：`Share a 2 sentence horror story`

## 完整 API 请求

LangDB 项目 URL 下的 `chat.completions.create`。

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/langdb/langdb.py` | `LangDB` |
