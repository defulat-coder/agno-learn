# save_conditional_steps.py — 实现原理分析

> 源文件：`cookbook/93_components/workflows/save_conditional_steps.py`

## 概述

本示例展示 Agno 的 **`Condition 步骤序列化`** 机制：Workflow 中的 Condition 容器根据 `evaluator` 函数的返回值决定是否执行内部步骤，该函数通过 Registry 按名注册和还原，实现动态条件分支的持久化。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `workflow.name` | `"Conditional Research Workflow"` | Workflow 名称 |
| `Condition.evaluator` | `is_tech_topic` | 条件判断函数（需 Registry） |
| `Condition.steps` | `[research_hackernews_step]` | 条件为 True 时执行的步骤 |
| `registry.functions` | `[is_tech_topic]` | 注册 evaluator 函数 |
| `hackernews_agent.tools` | `[HackerNewsTools()]` | HN 研究工具 |
| `web_agent.tools` | `[WebSearchTools()]` | 通用搜索工具 |

## 架构分层

```
用户代码层                      序列化/还原层
┌──────────────────────────┐    ┌─────────────────────────────────────┐
│ save_conditional_steps   │    │ 保存：                              │
│ .py                      │    │  Condition.to_dict()                │
│                          │    │   → {evaluator: "is_tech_topic"}    │
│ Condition(               │───>│                                      │
│   evaluator=is_tech_topic│    │ 还原：                              │
│   steps=[...hackernews]  │    │  Condition.from_dict(data, registry)│
│ )                        │    │   → registry.get_function(name)     │
└──────────────────────────┘    └─────────────────────────────────────┘
```

## 核心组件解析

### Condition 容器

`Condition` 在 `agno.workflow.condition` 中定义，执行时调用 `evaluator(step_input)` 决定分支：

```python
from agno.workflow.condition import Condition

Condition(
    name="TechTopicCondition",
    description="Check if topic is tech-related for HackerNews research",
    evaluator=is_tech_topic,   # 接受 StepInput，返回 bool
    steps=[research_hackernews_step],  # evaluator 返回 True 时执行
)
```

### evaluator 函数签名

```python
from agno.workflow.types import StepInput

def is_tech_topic(step_input: StepInput) -> bool:
    """Returns True to execute the conditional steps, False to skip."""
    topic = step_input.input or step_input.previous_step_content or ""
    tech_keywords = ["ai", "machine learning", "programming", "software", "tech"]
    is_tech = any(keyword in topic.lower() for keyword in tech_keywords)
    print(f"Condition: Topic is {'tech' if is_tech else 'not tech'}")
    return is_tech
```

### 序列化存储

```python
# Condition.to_dict()
{
    "type": "condition",
    "name": "TechTopicCondition",
    "evaluator": "is_tech_topic",  # 函数名字符串
    "steps": [
        {"name": "ResearchHackerNews", "agent_id": "..."}
    ]
}
```

### 完整工作流程序

```
步骤1: TechTopicCondition
  └── is_tech_topic(input) == True?
       ├── True  → ResearchHackerNews（hackernews_agent）
       └── False → 跳过此步骤

步骤2: ResearchWeb（web_agent，始终执行）

步骤3: WriteContent（content_agent，始终执行）
```

### Condition vs Loop vs Router

| 容器类型 | 判断函数 | 函数签名 | 行为 |
|---------|---------|---------|------|
| `Condition` | `evaluator` | `(StepInput) → bool` | True 执行内部步骤，False 跳过 |
| `Loop` | `end_condition` | `(List[StepOutput]) → bool` | True 停止循环，False 继续 |
| `Router` | `selector` | `(StepInput) → List[Step]` | 返回要执行的步骤列表 |

三者均通过 Registry `functions` 字段注册，按函数名还原。

## System Prompt 组装

各 Agent 在执行时组装 system prompt：

| Agent | instructions | tools |
|-------|-------------|-------|
| hackernews_agent | `"Research tech news and trends from Hacker News"` | HackerNewsTools |
| web_agent | `"Research general information from the web"` | WebSearchTools |
| content_agent | `"Create well-structured content from research data"` | 无 |

## 完整 API 请求

```python
# 输入为技术话题时，执行 ResearchHackerNews：
client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "Research tech news and trends from Hacker News"},
        {"role": "user", "content": "Latest AI developments in machine learning"}
    ],
    tools=[{"type": "function", "function": {"name": "get_top_hackernews_stories", ...}}],
    stream=True,
)

# 输入为非技术话题时，跳过 ResearchHackerNews，直接执行 ResearchWeb：
client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "Research general information from the web"},
        {"role": "user", "content": "非技术话题"}
    ],
    tools=[{"type": "function", "function": {"name": "web_search", ...}}],
    stream=True,
)
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["workflow.save(db)"] --> B["Condition.to_dict()<br/>evaluator → 函数名字符串"]
    B --> C["db.upsert_config()"]

    D["get_workflow_by_id(db, id, registry)"] --> E["Condition.from_dict(data, registry)"]
    E --> F["registry.get_function('is_tech_topic')"]
    F --> G["Condition（evaluator 已还原）"]

    G --> H["Workflow 执行<br/>step_input = 用户输入"]
    H --> I{is_tech_topic(step_input)?}
    I -- True --> J["ResearchHackerNews<br/>hackernews_agent"]
    I -- False --> K["跳过"]
    J --> L["ResearchWeb<br/>web_agent"]
    K --> L
    L --> M["WriteContent<br/>content_agent"]

    style A fill:#e1f5fe
    style M fill:#e8f5e9
    style I fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/workflow/condition.py` | `Condition` | 条件分支容器类 |
| `agno/workflow/condition.py` | `Condition.to_dict()` | evaluator → 函数名 |
| `agno/workflow/condition.py` | `Condition.from_dict()` | 函数名 → callable 还原 |
| `agno/workflow/types.py` | `StepInput` | evaluator 函数参数类型 |
| `agno/registry/registry.py` | `get_function()` L81 | 按名还原 evaluator |
