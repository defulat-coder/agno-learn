# 04_agent_os.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
AgentOS: Serving Knowledge via API
====================================
AgentOS wraps your agents and knowledge instances in a FastAPI server,
exposing them as API endpoints. This is how you move from a script to
a running service.

Key concepts:
- Multiple Knowledge instances can share the same vector_db and contents_db
- Each instance is identified by its `name` property
- Content is isolated per instance via the `linked_to` field
- AgentOS exposes /knowledge endpoints for managing content

Setup:
1. Run Qdrant: ./cookbook/scripts/run_qdrant.sh
2. pip install uvicorn

See also: 03_multi_tenant.py for tenant isolation patterns.
"""

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.knowledge.embedder.openai import OpenAIEmbedder
from agno.knowledge.knowledge import Knowledge
from agno.models.openai import OpenAIResponses
from agno.os import AgentOS
from agno.vectordb.qdrant import Qdrant
from agno.vectordb.search import SearchType

# ---------------------------------------------------------------------------
# Shared Infrastructure
# ---------------------------------------------------------------------------

qdrant_url = "http://localhost:6333"

vector_db = Qdrant(
    collection="agent_os_demo",
    url=qdrant_url,
    search_type=SearchType.hybrid,
    embedder=OpenAIEmbedder(id="text-embedding-3-small"),
)

contents_db = SqliteDb(db_file="tmp/agent_os.db")

# ---------------------------------------------------------------------------
# Knowledge Instances
# ---------------------------------------------------------------------------

# Each instance has a unique name — content is isolated via linked_to
company_knowledge = Knowledge(
    name="Company Docs",
    description="Internal company documentation",
    vector_db=vector_db,
    contents_db=contents_db,
)

product_knowledge = Knowledge(
    name="Product FAQ",
    description="Product frequently asked questions",
    vector_db=vector_db,
    contents_db=contents_db,
)

# ---------------------------------------------------------------------------
# Agents
# ---------------------------------------------------------------------------

support_agent = Agent(
    name="Support Agent",
    model=OpenAIResponses(id="gpt-5.2"),
    knowledge=company_knowledge,
    search_knowledge=True,
    markdown=True,
)

product_agent = Agent(
    name="Product Agent",
    model=OpenAIResponses(id="gpt-5.2"),
    knowledge=product_knowledge,
    search_knowledge=True,
    markdown=True,
)

# ---------------------------------------------------------------------------
# AgentOS
# ---------------------------------------------------------------------------

agent_os = AgentOS(
    agents=[support_agent, product_agent],
)
app = agent_os.get_app()

# ---------------------------------------------------------------------------
# Run
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    # Serves a FastAPI app. Use reload=True for local development.
    agent_os.serve(app="04_agent_os:app", reload=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/07_knowledge/03_production/04_agent_os.py`

## 概述

本示例展示 **AgentOS + 多 Knowledge 实例**：`company_knowledge` 与 `product_knowledge` 共享 `vector_db` 与 `contents_db`，通过 `name` / `linked_to` 区分内容；`AgentOS` 暴露 FastAPI 服务与 `/knowledge` 管理能力。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `company_knowledge` | `Knowledge(name="Company Docs", description=..., ...)` | 内部文档 |
| `product_knowledge` | `Knowledge(name="Product FAQ", ...)` | FAQ |
| `support_agent` | `Agent(name="Support Agent", model=OpenAIResponses(gpt-5.2), knowledge=company_knowledge, search_knowledge=True, markdown=True)` | 支持 |
| `product_agent` | 同上，绑定 `product_knowledge` | 产品 |
| `agent_os` | `AgentOS(agents=[support_agent, product_agent])` | OS 外壳 |
| `serve` | `agent_os.serve(app="04_agent_os:app", reload=True)` | 本地服务 |

## 架构分层

```
用户 / HTTP 客户端
        │
        ▼
   AgentOS.get_app() → FastAPI
        │ 路由到 agents / knowledge API
        ▼
   Agent._run → OpenAIResponses
```

## 核心组件解析

### AgentOS

将多个 Agent 与 Knowledge 注册到同一进程服务，便于从脚本升级为 API 形态（见 `agno/os`）。

### 运行机制与因果链

1. **路径**：HTTP 或脚本侧触发 `run` → 与独立脚本相同的 Agent 消息与模型调用。
2. **状态**：`contents_db` 持久化摄入记录；服务进程内驻留 Agent 实例。
3. **分支**：不同 Agent 绑定不同 `Knowledge`，检索范围不同。
4. **差异**：相对 `03_multi_tenant.py`，本示例强调 **HTTP 服务化** 而非仅租户标志。

## System Prompt 组装

两 Agent 均 `markdown=True`，无额外 `instructions`；`name` 默认不加入 context（`add_name_to_context` 未设）。

### 还原后的完整 System 文本（单 Agent 默认路径）

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

若启用 `add_name_to_context`，则会多出 “Your name is: Support Agent” 等，本文件未启用。

## 完整 API 请求

- **LLM**：`OpenAIResponses` → `responses.create`（`agno/models/openai/responses.py`）。
- **AgentOS HTTP**：由 FastAPI 路由封装对内部 `Agent` 的调用，非直接 OpenAI 裸 HTTP。

## Mermaid 流程图

```mermaid
flowchart TD
    subgraph Svc["AgentOS 服务"]
        A["【关键】AgentOS.get_app()"] --> B["路由到 Agent"]
    end
    B --> C["get_system_message"]
    C --> D["【关键】OpenAIResponses"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/os/__init__.py` / `AgentOS` | 应用组装 |
| `agno/agent/_messages.py` | System 与 run 消息 |
| `agno/models/openai/responses.py` | Responses API |
