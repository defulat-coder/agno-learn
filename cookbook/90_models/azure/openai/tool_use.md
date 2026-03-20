# tool_use.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Run `pip install ddgs` to install dependencies."""

from agno.agent import Agent
from agno.models.azure import AzureOpenAI
from agno.tools.websearch import WebSearchTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=AzureOpenAI(id="gpt-4o-mini"),
    tools=[WebSearchTools()],
    markdown=True,
)

agent.print_response("Whats happening in France?", stream=True)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/azure/openai/tool_use.py`

## 概述

**AzureOpenAI(gpt-4o-mini) + WebSearchTools**，流式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureOpenAI(id="gpt-4o-mini")` | 低成本 |
| `tools` | `[WebSearchTools()]` | 搜索 |
| `markdown` | `True` | Markdown |

## System Prompt 组装

### 还原后的完整 System 文本

```text
Use markdown to format your answers.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["tools"] --> B["【关键】Chat Completions tools"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/openai/like.py` | `invoke` | 请求 |
