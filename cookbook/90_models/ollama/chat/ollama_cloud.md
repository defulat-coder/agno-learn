# ollama_cloud.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""To use Ollama Cloud, you need to set the OLLAMA_API_KEY environment variable. Host is set to https://ollama.com by default."""

from agno.agent import Agent
from agno.models.ollama import Ollama

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

agent = Agent(
    model=Ollama(id="gpt-oss:120b-cloud"),
)

agent.print_response("What is the capital of France?", stream=True)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/ollama/chat/ollama_cloud.py`

## 概述

**Ollama Cloud**：`Ollama(id="gpt-oss:120b-cloud")`，依赖 `OLLAMA_API_KEY`，默认 host 指向云端（见模型文档）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Ollama(id="gpt-oss:120b-cloud")` | 无显式 `markdown`（默认 True） |

用户消息：`"What is the capital of France?"`，`stream=True`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["OLLAMA_API_KEY"] --> B["【关键】云端 chat 端点"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/ollama/chat.py` | `Ollama` 客户端 host |
