# structured_io_team.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/structured_io/structured_io_team.py`

## 概述

本示例展示 Agno Workflow **基于 Team 的结构化输入/输出流水线**：每个 Step 使用 `Team(output_schema=PydanticModel)` 输出结构化数据，后续 Step 的 Team 通过 `previous_step_content` 接收前一步的结构化结果，实现三阶段（研究→策略→计划）端到端的结构化数据流。

**核心配置：**

| Team | `output_schema` | 说明 |
|------|----------------|------|
| `AI Research Team` | `ResearchFindings` | 研究发现（洞察 + 置信度） |
| `Content Strategy Team` | `ContentStrategy` | 内容策略（受众 + 话题 + 日程） |
| `Content Planning Team` | `FinalContentPlan` | 最终计划（预算 + 时间线 + 风险） |

## 核心组件解析

### Pydantic 结构化输出模型

```python
class ResearchFindings(BaseModel):
    topic: str
    key_insights: List[str]
    trending_technologies: List[str]
    confidence_score: float  # 0.0-1.0

class ContentStrategy(BaseModel):
    target_audience: str
    content_pillars: List[str]
    hashtags: List[str]

class FinalContentPlan(BaseModel):
    campaign_name: str
    content_calendar: List[str]
    budget_estimate: str
    timeline: str
```

### 多 Team 结构化数据流

```python
research_team = Team(
    members=[research_specialist, data_analyst],
    output_schema=ResearchFindings,   # Step 1 输出结构化研究结果
)

strategy_team = Team(
    members=[content_strategist, marketing_expert],
    output_schema=ContentStrategy,    # Step 2 消费 Step 1 输出，产出策略
)

structured_workflow = Workflow(
    steps=[
        Step(name="Research Insights", team=research_team),
        Step(name="Content Strategy", team=strategy_team),
        Step(name="Final Planning", team=planning_team),
    ],
)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/team/team.py` | `Team.output_schema` | Team 结构化输出 |
| `agno/workflow/types.py` | `StepInput.previous_step_content` | 接收上步 Pydantic 输出 |
