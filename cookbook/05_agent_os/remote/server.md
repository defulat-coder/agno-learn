# server.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
AgentOS Server for Cookbook Client Examples
"""

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.knowledge.embedder.openai import OpenAIEmbedder
from agno.knowledge.knowledge import Knowledge
from agno.models.openai import OpenAIChat
from agno.os import AgentOS
from agno.team.team import Team
from agno.tools.calculator import CalculatorTools
from agno.tools.websearch import WebSearchTools
from agno.vectordb.chroma import ChromaDb
from agno.workflow.step import Step
from agno.workflow.workflow import Workflow

# ---------------------------------------------------------------------------
# Create Example
# ---------------------------------------------------------------------------

# =============================================================================
# Database Configuration
# =============================================================================

# SQLite database for sessions, memory, and content metadata
db = SqliteDb(id="cookbook-client-db", db_file="tmp/cookbook_client.db")

# =============================================================================
# Knowledge Base Configuration
# =============================================================================

knowledge = Knowledge(
    vector_db=ChromaDb(
        path="tmp/cookbook_chromadb",
        collection="cookbook_knowledge",
        embedder=OpenAIEmbedder(id="text-embedding-3-small"),
    ),
    contents_db=db,  # Required for content upload/management endpoints
)

# =============================================================================
# Agent Configuration
# =============================================================================

# Agent 1: Assistant with calculator tools and memory
assistant = Agent(
    name="Assistant",
    id="assistant-agent",
    description="You are a helpful AI assistant.",
    model=OpenAIChat(id="gpt-5.2"),
    db=db,
    instructions=[
        "You are a helpful AI assistant.",
        "Use the calculator tool for any math operations.",
        "You have access to a knowledge base - search it when asked about documents.",
    ],
    markdown=True,
    update_memory_on_run=True,  # Required for 03_memory_operations
    tools=[CalculatorTools()],
    knowledge=knowledge,
    search_knowledge=True,
)

# Agent 2: Researcher with web search capabilities
researcher = Agent(
    name="Researcher",
    id="researcher-agent",
    model=OpenAIChat(id="gpt-5"),
    db=db,
    instructions=[
        "You are a research assistant.",
        "Search the web for information when needed.",
        "Provide well-researched, accurate responses.",
    ],
    markdown=True,
    tools=[WebSearchTools()],
)

# =============================================================================
# Team Configuration
# =============================================================================

research_team = Team(
    name="Research Team",
    id="research-team",
    model=OpenAIChat(id="gpt-5.2"),
    members=[assistant, researcher],
    instructions=[
        "You are a research team that coordinates multiple specialists.",
        "Delegate math questions to the Assistant.",
        "Delegate research questions to the Researcher.",
        "Combine insights from team members for comprehensive answers.",
    ],
    markdown=True,
    db=db,
)

# =============================================================================
# Workflow Configuration
# =============================================================================

qa_workflow = Workflow(
    name="QA Workflow",
    description="A simple Q&A workflow that uses the assistant agent",
    id="qa-workflow",
    db=db,
    steps=[
        Step(
            name="Answer Question",
            agent=assistant,
        ),
    ],
)

# =============================================================================
# AgentOS Configuration
# =============================================================================

agent_os = AgentOS(
    id="cookbook-client-server",
    description="AgentOS server for running cookbook client examples",
    agents=[assistant, researcher],
    teams=[research_team],
    workflows=[qa_workflow],
    knowledge=[knowledge],
)

# FastAPI app instance (for uvicorn)
app = agent_os.get_app()

# =============================================================================
# Main Entry Point
# =============================================================================

# ---------------------------------------------------------------------------
# Run Example
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    agent_os.serve(app="server:app", reload=True, access_log=True, port=7778)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/05_agent_os/remote/server.py`

## 概述

本示例为 **远端 AgentOS 样板（7778）**：`assistant`、`researcher`、`research-team`、`qa-workflow`，Chroma 知识库；**无** `a2a_interface`，供 `RemoteAgent`/`RemoteTeam`/`RemoteWorkflow` 走 **标准 AgentOS HTTP**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `agent_os` | `id=cookbook-client-server` |  |
| 实体 | agents + teams + workflows + knowledge | 全类型 |

## System Prompt 组装

Team `instructions` 见 L89-94；各 Agent 见 L53-77。

## Mermaid 流程图

```mermaid
flowchart TD
    A["server.py :7778"] --> B["【关键】Remote* 客户端目标"]
    B --> C["本地 Agent/Team/Workflow.run"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/os` | `AgentOS` | 服务 |
