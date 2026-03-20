# set_client.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Run `uv pip install yfinance` to install dependencies."""

from agno.agent import Agent, RunOutput  # noqa
from agno.models.ollama import Ollama
from ollama import Client as OllamaClient

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=Ollama(id="llama3.1:8b", client=OllamaClient()),
    markdown=True,
)

# Print the response in the terminal
agent.print_response("Share a 2 sentence horror story")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/ollama/chat/set_client.py`

## 概述

**注入显式 `ollama.Client`**：`Ollama(id="llama3.1:8b", client=OllamaClient())`，使用默认本地 Ollama 服务。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Ollama(..., client=OllamaClient())` | 自定义客户端实例 |
| `markdown` | `True` | 默认 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["OllamaClient()"] --> B["【关键】绕过默认 client 工厂"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/ollama/chat.py` | `get_client` / `client` 字段 |
