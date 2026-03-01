# structured_io_function.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/structured_io/structured_io_function.py`

## 概述

本示例展示 Agno Workflow **函数 executor 返回 Pydantic BaseModel 实例作为 `StepOutput.content`**：函数步骤可以返回 `StepOutput(content=pydantic_instance)` 而非字符串，下一步骤通过 `step_input.previous_step_content` 接收到原始 Pydantic 对象（未被序列化），实现函数步骤之间类型安全的结构化数据传递。

**核心配置一览：**

| 模式 | executor 返回 | 下步 `previous_step_content` 类型 |
|------|-------------|-------------------------------|
| 字符串输出 | `StepOutput(content="text")` | `str` |
| Pydantic 输出 | `StepOutput(content=AnalysisReport(...))` | `AnalysisReport`（原始对象） |
| 字典输出 | `StepOutput(content={"key": "val"})` | `dict` |

## 核心组件解析

### 函数返回 Pydantic 实例

```python
def enhanced_analysis_function(step_input: StepInput) -> StepOutput:
    previous = step_input.previous_step_content  # 接收上步 ResearchFindings 实例

    if isinstance(previous, ResearchFindings):
        # 访问类型安全的字段
        key_findings = [f"Topic: {previous.topic}", f"Confidence: {previous.confidence_score}"]

    analysis_report = AnalysisReport(
        analysis_type="Structured Data Flow Analysis",
        structured_data_detected=isinstance(previous, ResearchFindings),
        key_findings=key_findings,
        confidence_score=previous.confidence_score if isinstance(previous, ResearchFindings) else 0.6,
    )

    return StepOutput(content=analysis_report, success=True)  # 返回 Pydantic 实例
```

### 下一步骤接收 Pydantic 实例

```python
def simple_data_processor(step_input: StepInput) -> StepOutput:
    if isinstance(step_input.previous_step_content, AnalysisReport):
        report = step_input.previous_step_content  # 类型安全的 AnalysisReport
        summary = {
            "confidence": report.confidence_score,
            "findings_count": len(report.key_findings),
        }
        return StepOutput(content=summary, success=True)
```

### 两个 Workflow 对比

```python
# structured_workflow: Agent → Function → Agent → Agent
structured_workflow = Workflow(
    steps=[research_step, analysis_step, strategy_step, planning_step],
)

# enhanced_workflow: Agent → Function(Pydantic出) → Function(Pydantic入) → Agent → Agent
enhanced_workflow = Workflow(
    steps=[research_step, enhanced_analysis_step, processor_step, strategy_step, planning_step],
)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/types.py` | `StepOutput.content` | 支持 str/dict/Pydantic 实例 |
| `agno/workflow/types.py` | `StepInput.previous_step_content` | 原样传递前驱步骤的 content |
