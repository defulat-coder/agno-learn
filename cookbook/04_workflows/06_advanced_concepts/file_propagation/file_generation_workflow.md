# file_generation_workflow.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/file_propagation/file_generation_workflow.py`

## 概述

本示例展示 Agno Workflow **文件生成与跨步骤自动传播**机制：第一步 Agent 使用 `FileGenerationTools` 生成 PDF 文件，Workflow 自动将生成的文件传播到下一步 Agent，第二步 Agent 接收并分析该 PDF，实现基于文件的跨步骤数据流。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| 文件生成工具 | `FileGenerationTools(enable_pdf_generation=True)` | 生成 PDF 工具 |
| 文件传播 | Workflow 自动传播 | 步骤 1 生成的文件自动传到步骤 2 |
| 最终输出 | `result.files` | 流水线末端的文件列表 |

## 核心组件解析

### 文件生成步骤

```python
from agno.tools.file_generation import FileGenerationTools

report_generator = Agent(
    name="Report Generator",
    tools=[FileGenerationTools(enable_pdf_generation=True)],   # 启用 PDF 生成
    instructions=[
        "When asked to create a report, use the generate_pdf_file tool to create it.",
    ],
)

generate_report_step = Step(name="Generate Report", agent=report_generator)
```

### 文件分析步骤（自动接收文件）

```python
report_analyzer = Agent(
    name="Report Analyzer",
    instructions=["Analyze the attached PDF report and provide insights."],
)

analyze_report_step = Step(name="Analyze Report", agent=report_analyzer)
```

### Workflow 自动文件传播

```python
report_workflow = Workflow(
    steps=[generate_report_step, analyze_report_step],
    db=SqliteDb(db_file="tmp/file_propagation_workflow.db"),
)

result = report_workflow.run(
    input="Create a Q4 2024 quarterly sales report and then analyze it.",
)

# 检查文件传播结果
if result.files:
    for f in result.files:
        print(f"File: {f.filename} ({f.mime_type}, {f.size} bytes)")
```

## 文件传播机制

```
Step 1 (generate_report):
  agent 调用 generate_pdf_file 工具 → 生成 PDF
  Workflow 捕获步骤输出中的文件对象

Step 2 (analyze_report):
  Workflow 将 Step 1 的文件自动注入到 step_input.files
  agent 接收文件作为附件分析
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/tools/file_generation.py` | `FileGenerationTools` | PDF 等文件生成工具 |
| `agno/workflow/types.py` | `StepInput.files` | 传入步骤的文件列表 |
| `agno/run/workflow.py` | `WorkflowRunOutput.files` | 最终输出的文件列表 |
