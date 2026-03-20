# basic_reasoning_stream.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Basic Reasoning Stream
======================

Demonstrates this reasoning cookbook example.
"""

from agno.agent import Agent
from agno.models.anthropic import Claude
from agno.run.agent import RunEvent  # noqa


# ---------------------------------------------------------------------------
# Create Example
# ---------------------------------------------------------------------------
def run_example() -> None:
    # Create an agent with reasoning enabled
    agent = Agent(
        reasoning_model=Claude(
            id="claude-sonnet-4-5",
            thinking={"type": "enabled", "budget_tokens": 1024},
        ),
        reasoning=True,
        instructions="Think step by step about the problem.",
    )

    prompt = "What is 25 * 37? Show your reasoning."

    agent.print_response(prompt, stream=True, stream_events=True)

    # # or you can capture the event using
    # for run_output_event in agent.run(
    #     prompt,
    #     stream=True,
    #     stream_events=True,
    # ):
    #     if run_output_event.event == RunEvent.run_started:
    #         print(f"\nEVENT: {run_output_event.event}")

    #     elif run_output_event.event == RunEvent.reasoning_started:
    #         print(f"\nEVENT: {run_output_event.event}")
    #         print("Reasoning started...\n")

    #     elif run_output_event.event == RunEvent.reasoning_content_delta:
    #         # This is the NEW streaming event for reasoning content
    #         # It streams the raw content as it's being generated
    #         print(run_output_event.reasoning_content, end="", flush=True)

    #     elif run_output_event.event == RunEvent.run_content:
    #         if run_output_event.content:
    #             print(run_output_event.content, end="", flush=True)

    #     elif run_output_event.event == RunEvent.run_completed:
    #         print(f"\n\nEVENT: {run_output_event.event}")


# ---------------------------------------------------------------------------
# Run Example
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    run_example()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/10_reasoning/models/anthropic/basic_reasoning_stream.py`

## 概述

本示例展示 Agno 的 **内置推理（reasoning）**、**独立 reasoning_model** 机制：通过 `Agent` 配置模型与可选推理链路，完成示例任务。
**核心配置一览（首个/主要 Agent）：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `instructions` | `'Think step by step about the problem.'` | 显式参数 |
| `reasoning` | `True` | 显式参数 |
| `reasoning_model` | `Claude(id='claude-sonnet-4-5', thinking={'type': 'enabled', 'budget_tokens': 1024})` | 显式参数 |
| `db` | `None` | 未设置 |
| `knowledge` | `None` | 未设置 |
| `tools` | `None` | 未设置 |
| `description` | `None` | 未设置 |
| `system_message` | `None` | 未设置 |

## 架构分层

```
用户代码层                agno.agent 层
┌──────────────────┐    ┌──────────────────────────────────┐
│ basic_reasoning_st│    │ Agent._run / _arun               │
│ Agent(...)       │───>│  get_system_message()            │
│ print_response   │    │  get_run_messages()              │
└──────────────────┘    │  handle_reasoning（若启用）       │
                        └──────────────────────────────────┘
                                │
                                ▼
                        ┌──────────────┐
                        │ Model.invoke │
                        └──────────────┘
```

## 核心组件解析

### Agent 与推理

当 `reasoning=True` 或配置了 `reasoning_model` 时，`_run` 在适当时机调用 `handle_reasoning` / `handle_reasoning_stream`（`agno/agent/_response.py` L72+），内部进入 `reason()` 生成推理内容。

### 运行机制与因果链

1. **路径**：`Agent.run` / `print_response` → `get_system_message` → `get_run_messages` → 模型；推理启用时插入推理阶段。
2. **状态**：默认不写 session/db；若配置 `db`、`update_memory_on_run` 等则会持久化（见各文件）。
3. **分支**：仅主模型 vs `reasoning_model`、流式 `stream=True`、`show_full_reasoning` 影响输出展示。
4. **定位**：位于 `cookbook` 的 **推理/模型变体** 主题，对照同目录示例可看出参数差异。

## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 1 | `description` | 未设置 | 否 |
| 2 | `role` | 未设置 | 否 |
| 3 | `instructions` | 'Think step by step about the problem.' | 是 |
| 4.1 | `markdown` | False | 否 |

### 拼装顺序与源码锚点

默认：`# 1` 自定义 `system_message` 早退 → 否则 `# 2` build_context → `# 3.1` instructions → `# 3.2.1` markdown 附加段 → `# 3.3.1-3.3.4` description/role/instructions 拼接…（`agno/agent/_messages.py`）。

### 还原后的完整 System 文本

```text
Think step by step about the problem.
```

### 段落释义（模型视角）

- `instructions`：直接约束角色与解题步骤（若存在）。
- `markdown`：附加格式化要求，减少非结构化输出。
- 无业务 instructions 时，模型仅受默认附加段约束，行为更依赖用户消息。

### 与 User / Developer 消息的边界

用户任务来自 `print_response`/`run` 的输入字符串；OpenAI Chat 适配器使用 `messages` 列表中的 `system`/`user`/`assistant` 角色。

## 完整 API 请求

```python
# OpenAIChat（常见）：
# client.chat.completions.create(
#     model=<id>,
#     messages=[...],  # system + user + ...
#     **get_request_params(...),
# )
# 见 agno/models/openai/chat.py invoke() L385+
```

> 其他厂商请对照对应 `Model` 子类的 `invoke`/`ainvoke`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["用户代码 print_response"] --> B["Agent._run"]
    B --> C["get_system_message()"]
    B --> D["【关键】handle_reasoning / stream"]
    D --> E["get_run_messages"]
    E --> F["【关键】Model.invoke"]
    F --> G["RunOutput / 流式输出"]
```

- **【关键】handle_reasoning / stream**：`reasoning` 或 `reasoning_model` 启用时进入推理子流程。
- **【关键】Model.invoke**：向提供商发起 chat completion / responses 等请求。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `get_system_message` L799+ | 委托 `_messages` |
| `agno/agent/_messages.py` | `get_system_message()` L106+ | system 拼装 |
| `agno/agent/_response.py` | `handle_reasoning()` L72+ | 推理处理 |
| `agno/agent/_run.py` | run 主循环 | 调用推理与模型 |
| `agno/models/openai/chat.py` | `invoke()` L385+ | Chat Completions |
