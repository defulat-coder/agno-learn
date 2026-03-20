# async_mongodb_for_workflow.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

````python
"""
Run: `pip install openai ddgs pymongo motor` to install dependencies
Run: `python cookbook/db/mongo/async_mongo/async_mongodb_for_workflow.py` to run the workflow

Run a local MongoDB server using:
```bash
docker run -d \
  --name local-mongo \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  mongo
```
or use our script:
```bash
./scripts/run_mongodb.sh
```
"""

import asyncio

from agno.agent import Agent
from agno.db.mongo import AsyncMongoDb
from agno.models.openai import OpenAIChat
from agno.team import Team
from agno.tools.hackernews import HackerNewsTools
from agno.tools.websearch import WebSearchTools
from agno.workflow.step import Step
from agno.workflow.workflow import Workflow

# ---------------------------------------------------------------------------
# Setup
# ---------------------------------------------------------------------------
db_url = "mongodb://mongoadmin:secret@localhost:27017"
db = AsyncMongoDb(db_url=db_url)

# ---------------------------------------------------------------------------
# Create Workflow
# ---------------------------------------------------------------------------
hackernews_agent = Agent(
    name="Hackernews Agent",
    model=OpenAIChat(id="gpt-4o-mini"),
    tools=[HackerNewsTools()],
    role="Extract key insights and content from Hackernews posts",
)
web_agent = Agent(
    name="Web Agent",
    model=OpenAIChat(id="gpt-4o-mini"),
    tools=[WebSearchTools()],
    role="Search the web for the latest news and trends",
)
content_planner = Agent(
    name="Content Planner",
    model=OpenAIChat(id="gpt-4o"),
    instructions=[
        "Plan a content schedule over 4 weeks for the provided topic and research content",
        "Ensure that I have posts for 3 posts per week",
    ],
)

research_team = Team(
    name="Research Team",
    members=[hackernews_agent, web_agent],
    instructions="Research tech topics from Hackernews and the web",
)

research_step = Step(
    name="Research Step",
    team=research_team,
)
content_planning_step = Step(
    name="Content Planning Step",
    agent=content_planner,
)
content_creation_workflow = Workflow(
    name="Content Creation Workflow",
    description="Automated content creation from blog posts to social media",
    db=db,
    steps=[research_step, content_planning_step],
)

# ---------------------------------------------------------------------------
# Run Workflow
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    asyncio.run(
        content_creation_workflow.aprint_response(
            input="AI trends in 2024",
            markdown=True,
        )
    )
````

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/06_storage/mongo/async_mongo/async_mongodb_for_workflow.py`

## 概述

本示例展示 **AsyncMongoDb** 作为 **Workflow** 的 `db`：`Workflow(..., db=...)` 持久化工作流会话；结构为 **Team 研究步 + Agent 规划步**（与 `postgres_for_workflow.py` 一致）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `research_team` | `Team(members=[...], instructions=...)` | 第一步 |
| `content_planner` | `OpenAIChat`, `instructions` 列表 | 第二步 |
| `Workflow` | `db=AsyncMongoDb(...)` | 会话表见源文件 |

## 架构分层

`Workflow.print_response` → 各 Step → Agent/Team run → DB。

## System Prompt 组装

分步还原见各 Agent/Team 源码；无单一 OS 级 system。

## 完整 API 请求

各步 `OpenAIChat.invoke` / Team 主循环。

## Mermaid 流程图

```mermaid
flowchart LR
    R["Research"] --> P["Planning"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/workflow/workflow.py` | `Workflow` |
| `agno/db/*` | `db` 参数 |
