# workflow_agent_with_condition.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/workflow_agent/workflow_agent_with_condition.py`

## 概述

本示例展示 Agno Workflow **`WorkflowAgent` 与 `Condition` 联合使用**：`WorkflowAgent` 作为智能控制器，根据对话历史决定是执行 Workflow 步骤还是直接从历史回答；同时结合 `Condition(evaluator=needs_editing)` 根据故事内容动态决定是否执行编辑步骤，实现会话感知的条件工作流。

**核心架构：**

```
WorkflowAgent（智能控制器）
  ↓ 根据历史判断
Workflow.steps:
  [1] write_step  → Story Writer Agent
  [2] Condition(needs_editing) → edit_step (可选)
  [3] format_step → Story Formatter Agent
  [4] add_references (函数 executor)
```

## 核心组件解析

### WorkflowAgent + Condition 组合

```python
from agno.workflow import WorkflowAgent

workflow_agent = WorkflowAgent(model=OpenAIChat(id="gpt-5.2"), num_history_runs=4)

workflow = Workflow(
    agent=workflow_agent,           # 智能控制器
    steps=[
        write_step,
        Condition(
            name="editing_condition",
            evaluator=needs_editing,    # 根据故事长度/标点决定是否编辑
            steps=[edit_step],
        ),
        format_step,
        add_references,             # 函数 executor：追加引用
    ],
    db=PostgresDb(db_url),          # 生产级 PostgreSQL 存储
)
```

### 条件判断函数

```python
def needs_editing(step_input: StepInput) -> bool:
    story = step_input.previous_step_content or ""
    word_count = len(story.split())
    # 超过 50 词或含特殊标点则触发编辑
    return word_count > 50 or any(punct in story for punct in ["!", "?", ";", ":"])
```

### 多轮对话行为

```
第 1 轮: "Tell me a story about a brave knight" → 执行所有步骤
第 2 轮: "What was the knight's name?" → WorkflowAgent 直接从历史回答（无需重新执行步骤）
第 3 轮: "Now tell me about a cat" → 执行所有步骤
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/agent.py` | `WorkflowAgent` | 会话感知的智能控制器 |
| `agno/workflow/condition.py` | `Condition` | 条件分支执行 |
| `agno/db/postgres.py` | `PostgresDb` | 生产级会话持久化 |
