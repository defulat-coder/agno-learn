# background_evals_example.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Example: Per-Hook Background Control with AgentAsJudgeEval in AgentOS

This example demonstrates fine-grained control over which hooks run in background:
- Set eval.run_in_background = True for eval instances
- AgentAsJudgeEval evaluates output quality based on custom criteria
"""

from agno.agent import Agent
from agno.db.sqlite import AsyncSqliteDb
from agno.eval.agent_as_judge import AgentAsJudgeEval
from agno.models.openai import OpenAIChat
from agno.os import AgentOS

# ---------------------------------------------------------------------------
# Create Example
# ---------------------------------------------------------------------------

# Setup database
db = AsyncSqliteDb(db_file="tmp/agent_as_judge_evals.db")

# AgentAsJudgeEval for completeness - runs synchronously (blocks response)
completeness_eval = AgentAsJudgeEval(
    db=db,
    name="Completeness Check",
    model=OpenAIChat(id="gpt-5.2"),
    criteria="Response should be thorough, complete, and address all aspects of the question",
    print_results=True,
    print_summary=True,
    telemetry=True,
)
# completeness_eval.run_in_background = False (default - blocks)

# AgentAsJudgeEval for quality - runs in background (non-blocking)
quality_eval = AgentAsJudgeEval(
    db=db,
    name="Quality Assessment",
    model=OpenAIChat(id="gpt-5.2"),
    criteria="Response should be well-structured, concise, and professional",
    scoring_strategy="numeric",
    threshold=8,
    additional_guidelines=[
        "Check if response is easy to understand",
        "Verify response is not overly verbose",
    ],
    print_results=True,
    print_summary=True,
    run_in_background=True,  # Run this eval as a background task
)

agent = Agent(
    id="geography-agent",
    name="GeographyAgent",
    model=OpenAIChat(id="gpt-5.2"),
    instructions="You are a helpful geography assistant. Provide accurate and concise answers.",
    db=db,
    post_hooks=[
        completeness_eval,  # run_in_background=False - runs first, blocks
        quality_eval,  # run_in_background=True - runs after response
    ],
    markdown=True,
    telemetry=False,
)

# Create AgentOS
agent_os = AgentOS(agents=[agent])
app = agent_os.get_app()

# Flow:
# 1. Agent processes request
# 2. Sync hooks run (completeness_eval)
# 3. Response sent to user
# 4. Background hooks run (quality_eval)

# Test with:
# curl -X POST http://localhost:7777/agents/geography-agent/runs \
#   -F "message=What is the capital of France?" -F "stream=false"

# ---------------------------------------------------------------------------
# Run Example
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    agent_os.serve(app="background_evals_example:app", port=7777, reload=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/05_agent_os/background_tasks/background_evals_example.py`

## 概述

本示例用 **`AgentAsJudgeEval`** 作为 **`post_hooks`**：一个 **`run_in_background=False`**（默认，阻塞至评测完再返回），另一个 **`run_in_background=True`**（响应后后台评测）。**`AsyncSqliteDb`**；主 Agent **`OpenAIChat(id="gpt-5.2")`**，指令为地理助手。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `post_hooks` | `[completeness_eval, quality_eval]` | 顺序执行；后者后台 |
| `completeness_eval.run_in_background` | False（默认） | 同步阻塞 |
| `quality_eval.run_in_background` | True | 非阻塞 |
| `Agent.instructions` | `"You are a helpful geography assistant..."` | 见源文件 |
| `telemetry` | `False`（Agent） | 关闭 Agent 遥测 |

## 运行机制与因果链

1. 请求 → Agent 生成回复 → **先跑 completeness**（阻塞）→ 返回用户 → **quality 后台**。  
2. `AsyncSqliteDb` 存评测结果。

## System Prompt 组装

### 还原后的完整 System 文本（主 Agent）

```text
You are a helpful geography assistant. Provide accurate and concise answers.

```

并含 markdown 与时间（`add_datetime` 未显式开启则为默认；本文件 `markdown=True` 故有 markdown 句）。

## 完整 API 请求

`OpenAIChat` → `chat.completions.create`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["Agent.run"] --> B["【关键】completeness 同步"]
    B --> C["返回用户"]
    C --> D["【关键】quality 后台"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/agent_as_judge.py` | `AgentAsJudgeEval` |
