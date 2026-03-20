# agent_as_judge_binary.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Binary Agent-as-Judge Evaluation
================================

Demonstrates pass/fail response quality evaluation.
"""

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.eval.agent_as_judge import AgentAsJudgeEval
from agno.models.openai import OpenAIChat

# ---------------------------------------------------------------------------
# Create Database
# ---------------------------------------------------------------------------
db = SqliteDb(db_file="tmp/agent_as_judge_binary.db")

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    instructions="You are a customer service agent. Respond professionally.",
    db=db,
)

# ---------------------------------------------------------------------------
# Create Evaluation
# ---------------------------------------------------------------------------
evaluation = AgentAsJudgeEval(
    name="Professional Tone Check",
    criteria="Response must maintain professional tone without informal language or slang",
    db=db,
)

# ---------------------------------------------------------------------------
# Run Evaluation
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    response = agent.run("I need help with my account")
    result = evaluation.run(
        input="I need help with my account",
        output=str(response.content),
        print_results=True,
        print_summary=True,
    )
    print(f"Result: {'PASSED' if result.results[0].passed else 'FAILED'}")
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/09_evals/agent_as_judge/agent_as_judge_binary.py`

## 概述

本示例演示 **二元通过/失败** 式 Agent-as-Judge（未显式写 `scoring_strategy` 时由默认与 `criteria` 决定；脚本侧重 `result.results[0].passed`）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `agent.instructions` | 客服专业回复 | 被测 |
| `criteria` | 专业语气、无俚语 | 评判 |

### 还原 instructions

```text
You are a customer service agent. Respond professionally.
```

## 完整 API 请求

`chat.completions` 两轮：被测 + 评判。

## Mermaid 流程图

```mermaid
flowchart TD
    A["agent.run"] --> B["【关键】evaluation.run"]
    B --> P{{ "passed?" }}
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/eval/agent_as_judge.py` | 二元策略 |
