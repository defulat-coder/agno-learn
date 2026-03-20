# 01_basic_user_input.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Basic User Input HITL Example

This example demonstrates how to pause a workflow to collect user input
before executing a step. The user input is then available to the step
via step_input.additional_data["user_input"].

Use case: Collecting parameters from the user before processing data.

Two ways to define user_input_schema:
1. List of UserInputField objects (recommended) - explicit and type-safe
2. List of dicts - simple but less explicit
"""

from agno.agent import Agent
from agno.db.postgres import PostgresDb
from agno.models.openai import OpenAIChat
from agno.workflow.decorators import pause
from agno.workflow.step import Step
from agno.workflow.types import StepInput, StepOutput, UserInputField
from agno.workflow.workflow import Workflow


# Step 1: Analyze data (no HITL)
def analyze_data(step_input: StepInput) -> StepOutput:
    """Analyze the data and provide summary."""
    user_query = step_input.input or "data"
    return StepOutput(
        content=f"Analysis complete: Found 1000 records matching '{user_query}'. "
        "Ready for processing with user-specified parameters."
    )


# Step 2: Process with user-provided parameters (HITL - user input)
# Using UserInputField for schema - explicit and type-safe
@pause(
    name="Process Data",
    requires_user_input=True,
    user_input_message="Please provide processing parameters:",
    user_input_schema=[
        UserInputField(
            name="threshold",
            field_type="float",
            description="Processing threshold (0.0 to 1.0)",
            required=True,
        ),
        UserInputField(
            name="mode",
            field_type="str",
            description="Processing mode: 'fast' or 'accurate'",
            required=True,
        ),
        UserInputField(
            name="batch_size",
            field_type="int",
            description="Number of records per batch",
            required=False,
        ),
    ],
)
def process_with_params(step_input: StepInput) -> StepOutput:
    """Process data with user-provided parameters."""
    # Get user input from additional_data
    user_input = (
        step_input.additional_data.get("user_input", {})
        if step_input.additional_data
        else {}
    )

    threshold = user_input.get("threshold", 0.5)
    mode = user_input.get("mode", "fast")
    batch_size = user_input.get("batch_size", 100)

    previous = step_input.previous_step_content or ""

    return StepOutput(
        content=f"Processing complete!\n"
        f"- Input: {previous}\n"
        f"- Threshold: {threshold}\n"
        f"- Mode: {mode}\n"
        f"- Batch size: {batch_size}\n"
        f"- Records processed: 1000"
    )


# Step 3: Generate report (no HITL)
writer_agent = Agent(
    name="Report Writer",
    model=OpenAIChat(id="gpt-4o-mini"),
    instructions=[
        "You are a report writer.",
        "Given processing results, write a brief summary report.",
        "Keep it concise - 2-3 sentences.",
    ],
)


# Define steps
analyze_step = Step(name="analyze_data", executor=analyze_data)
process_step = Step(
    name="process_data", executor=process_with_params
)  # @pause auto-detected
report_step = Step(name="generate_report", agent=writer_agent)

# Create workflow
workflow = Workflow(
    name="data_processing_with_params",
    db=PostgresDb(db_url="postgresql+psycopg://ai:ai@localhost:5532/ai"),
    steps=[analyze_step, process_step, report_step],
)

if __name__ == "__main__":
    print("Starting data processing workflow...")
    print("=" * 50)

    run_output = workflow.run("customer transactions from Q4")

    # Handle HITL pauses
    while run_output.is_paused:
        # Show paused step info
        print(
            f"\n[PAUSED] Workflow paused at step {run_output.paused_step_index}: '{run_output.paused_step_name}'"
        )

        # Check for user input requirements
        for requirement in run_output.steps_requiring_user_input:
            print(f"\n[HITL] Step '{requirement.step_name}' requires user input")
            print(f"[HITL] {requirement.user_input_message}")

            # Display schema and collect input
            if requirement.user_input_schema:
                print("\nRequired fields:")
                user_values = {}
                for field in requirement.user_input_schema:
                    required_marker = "*" if field.required else ""
                    field_desc = f" - {field.description}" if field.description else ""
                    prompt = f"  {field.name}{required_marker} ({field.field_type}){field_desc}: "

                    value = input(prompt).strip()

                    # Convert to appropriate type
                    if value:
                        if field.field_type == "int":
                            user_values[field.name] = int(value)
                        elif field.field_type == "float":
                            user_values[field.name] = float(value)
                        elif field.field_type == "bool":
                            user_values[field.name] = value.lower() in (
                                "true",
                                "yes",
                                "1",
                            )
                        else:
                            user_values[field.name] = value

                # Set the user input
                requirement.set_user_input(**user_values)
                print("\n[HITL] User input received - continuing workflow...")

        # Check for confirmation requirements
        for requirement in run_output.steps_requiring_confirmation:
            print(f"\n[HITL] Step '{requirement.step_name}' requires confirmation")
            print(f"[HITL] {requirement.confirmation_message}")

            user_input = input("\nContinue? (yes/no): ").strip().lower()
            if user_input in ("yes", "y"):
                requirement.confirm()
            else:
                requirement.reject()

        # Continue the workflow
        run_output = workflow.continue_run(run_output)

    print("\n" + "=" * 50)
    print(f"Status: {run_output.status}")
    print(f"Output:\n{run_output.content}")
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/04_workflows/_07_human_in_the_loop/user_input/01_basic_user_input.py`

## 概述

本示例展示 Agno 的 **Workflow HITL：@pause 装饰器收集用户输入** 与 **Agent 步骤生成报告**：前两步为函数 Step（第二步暂停填表），第三步用 `Agent` 基于处理结果写摘要；用户输入经 `step_input.additional_data["user_input"]` 注入。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `Workflow.name` | `"data_processing_with_params"` | 工作流名 |
| `Workflow.db` | `PostgresDb(db_url="postgresql+psycopg://ai:ai@localhost:5532/ai")` | 需本地 PG |
| `Workflow.steps` | `analyze_step`, `process_step`, `report_step` | 分析 → HITL 处理 → Agent 报告 |
| `@pause` | `requires_user_input=True`，`user_input_message`，`user_input_schema`（3 个 `UserInputField`） | 装饰 `process_with_params` |
| `writer_agent` | `Agent(name="Report Writer", model=OpenAIChat(id="gpt-4o-mini"), instructions=[...])` | 报告生成 |
| `writer_agent.markdown` | `False`（默认） | 未启用 markdown 附加说明 |
| `writer_agent.tools` | 未设置 | 无工具 |

## 架构分层

```
用户代码层                agno.workflow + agno.agent
┌──────────────────┐    ┌──────────────────────────────────┐
│ workflow.run()   │───>│ Step: analyze → @pause 收集字段     │
│ set_user_input   │    │  → process_with_params              │
│ continue_run     │    │  → Step(agent=writer_agent)       │
│                  │    │     Agent._run → get_system_message │
│                  │    │     OpenAIChat.invoke → completions │
└──────────────────┘    └──────────────────────────────────┘
```

## 核心组件解析

### @pause 与 Step 绑定

`@pause(...)` 将 HITL 元数据挂到可调用对象上；`Step(name="process_data", executor=process_with_params)` 由框架识别为需用户输入的步骤，暂停时出现在 `steps_requiring_user_input`。

### UserInputField 与 additional_data

`process_with_params` 从 `step_input.additional_data["user_input"]` 读取 `threshold`/`mode`/`batch_size`，与 `UserInputField` 定义一致。

### Report Writer Agent

`writer_agent` 在 `generate_report` 步骤执行时走标准 Agent 管线：`get_system_message()`（`agno/agent/_messages.py` `L106` 起）拼装默认 system，再 `get_run_messages()` 组用户消息（含前序步骤输出）。

### 运行机制与因果链

1. **路径**：`run("customer transactions from Q4")` → `analyze_data` → 暂停 → CLI 填表 → `set_user_input` → `continue_run` → `process_with_params` → `writer_agent` 调用 LLM → 结束。
2. **状态**：`PostgresDb` 持久化工作流与 Agent 会话（若框架为步骤级 agent 建 session）。
3. **分支**：无确认；仅有 user input HITL。
4. **差异**：相对 `02_step_user_input.py`，本例用 **装饰器** 声明 HITL，而非 `Step(requires_user_input=...)`。

## System Prompt 组装

### 全局 Workflow

不存在单一「工作流级」system；**仅 `writer_agent` 这一步**产生 LLM system。

### Report Writer：`get_system_message()` 默认路径

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 1 | `system_message` | 未设置 | 否（走默认拼装） |
| 2 | `build_context` | 默认 `True` | 是 |
| 3 | `instructions` | 三句 list（见下「还原」） | 是（`# 3.1`，多行时为多条 `- ...`，`use_instruction_tags=False`） |
| 4 | `description` / `role` | 未设置 | 否 |
| 5 | `markdown` | `False` | 否（不追加「Use markdown...」） |
| 6 | `name` + `add_name_to_context` | `name` 有，`add_name_to_context` 默认 False | 否 |

### 拼装顺序与源码锚点

默认路径：`# 3.1` 收集 `instructions` → `# 3.3.3` 写入多条 bullet（`L241-255` `_messages.py`）→ 无 `# 3.3.4` 附加段（markdown 等未开）→ 返回 `Message(role=system, ...)`（具体 role 见 `agent.system_message_role` 默认）。

### 还原后的完整 System 文本

```text
- You are a report writer.
- Given processing results, write a brief summary report.
- Keep it concise - 2-3 sentences.

```

（若 `OpenAIChat` 的 `instructions` 非空，还会经 `# 3.1` 合并进上述块；本示例使用默认 `OpenAIChat(id="gpt-4o-mini")`，通常无额外 model 指令。）

### 段落释义（模型视角）

- 三条 bullet 约束角色为报告撰写者、输入为「处理结果」、输出长度控制在 2～3 句。

### 与 User 消息边界

用户消息为工作流传入的、包含前序步骤输出的运行输入；system 仅含上述指令，不含 HITL 表单字段（表单只进入 `process_with_params` 的 Step）。

## 完整 API 请求

`OpenAIChat` 使用 Chat Completions（`agno/models/openai/chat.py` `L412-417` `chat.completions.create`）。

```python
# 一次典型 Agent 调用（结构示意；messages 由 Agent 内部组装）
client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "<见上一节还原的 system 文本>"},
        {"role": "user", "content": "<含 process 步骤输出的用户消息>"},
    ],
)
```

> 与第 5 节 system 块对应；user 正文依赖运行时前序 `StepOutput`，无法仅从本文件静态固定。

## Mermaid 流程图

```mermaid
flowchart TD
    A["workflow.run"] --> B["analyze_data"]
    B --> C["【关键】@pause 收集 user_input"]
    C --> D["process_with_params"]
    D --> E["【关键】writer_agent LLM"]
    E --> F["完成"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/workflow/decorators.py` | `@pause` | 声明 HITL |
| `agno/agent/_messages.py` | `get_system_message()` L106+ | system 拼装 |
| `agno/models/openai/chat.py` | `OpenAIChat.invoke()` L385+ | Chat Completions |
