# level_1_tools.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Level 1: Agent with Tools
======================================
The simplest useful agent. A model, tools, and clear instructions.
No memory, no persistence — pure stateless tool calling.

This is where every agent should start. You'd be surprised how much
a single agent with good instructions and the right tools can accomplish.

Run standalone:
    python cookbook/levels_of_agentic_software/level_1_tools.py

Run via Agent OS:
    python cookbook/levels_of_agentic_software/run.py
    Then visit https://os.agno.com and select "L1 Coding Agent"

Example prompt:
    "Write a Fibonacci function, save it to fib.py, and run it to verify"
"""

from pathlib import Path

from agno.agent import Agent
from agno.models.openai import OpenAIResponses
from agno.tools.coding import CodingTools

# ---------------------------------------------------------------------------
# Workspace
# ---------------------------------------------------------------------------
WORKSPACE = Path(__file__).parent.joinpath("workspace")
WORKSPACE.mkdir(parents=True, exist_ok=True)

# ---------------------------------------------------------------------------
# Agent Instructions
# ---------------------------------------------------------------------------
instructions = """\
You are a coding agent. You write clean, well-documented Python code.

## Workflow

1. Understand the task
2. Write the code and save it to a file
3. Run the file to verify it works
4. If there are errors, fix them and re-run

## Rules

- Always save code to files before running
- Include type hints on function signatures
- Add a brief docstring to each function
- Test with at least 2-3 example inputs
- No emojis\
"""

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
l1_coding_agent = Agent(
    name="L1 Coding Agent",
    model=OpenAIResponses(id="gpt-5.2"),
    instructions=instructions,
    tools=[CodingTools(base_dir=WORKSPACE, all=True)],
    add_datetime_to_context=True,
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    l1_coding_agent.print_response(
        "Write a Fibonacci function that returns the nth Fibonacci number. "
        "Save it to fib.py with a main block that prints the first 10 values, "
        "then run it to verify.",
        stream=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/levels_of_agentic_software/level_1_tools.py`

## 概述

Level 1: Agent with Tools

本示例归类：**单 Agent**；模型相关类型：`OpenAIResponses`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | 'L1 Coding Agent' | `Agent(...)` |
| `model` | OpenAIResponses(id='gpt-5.2'…) | `Agent(...)` |
| `instructions` | 'You are a coding agent. You write clean, well-documented Python code.\n\n## Workflow\n\n1. Understand the task\n2. Write t...' | `Agent(...)` |
| `add_datetime_to_context` | True | `Agent(...)` |
| `markdown` | True | `Agent(...)` |
| （Model 类） | `OpenAIResponses` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ level_1_tools.py     │  ──▶  │ Agent → get_run_messages → Model │
└──────────────────────┘         └────────────────────────────────┘
                                          │
                                          ▼
                                  ┌───────────────┐
                                  │ 对应 Model 子类 │
                                  └───────────────┘
```

## 核心组件解析

### 运行机制与因果链

1. **入口**：从模块 `__main__` 或暴露的 `agent` / `team` 调用进入；同步用 `print_response` / `run`，异步用 `aprint_response` / `arun`（若源码中有）。
2. **消息**：默认路径下 system 内容由 `get_system_message()`（`libs/agno/agno/agent/_messages.py` 约 **L106** 起）按分段逻辑拼装；若显式传入 `system_message` 则早退使用该字符串。
3. **模型**：具体 HTTP/SDK 形态以 `libs/agno/agno/models/` 下对应类的 `invoke` / `ainvoke` 为准（勿默认写成单一 `chat.completions`）。
4. **副作用**：若配置 `db`、`knowledge`、`memory`，运行会读写存储；仅以本文件为准对照。

### 与框架的衔接

- **System**：`get_system_message()` 锚点 `agno/agent/_messages.py` **L106+**。
- **运行**：`Agent.print_response` 等入口 `agno/agent/agent.py`（以当前仓库检索为准）。

## System Prompt 组装

| 序号 | 组成部分 | 本文件 | 是否生效 |
|------|---------|--------|---------|
| 1 | `instructions` / `description` 等 | 见核心配置表与源码 | 有赋值则生效 |
| 2 | 默认分段（markdown、时间等） | 取决于 `Agent` 默认与显式参数 | 视参数 |

### 拼装顺序与源码锚点

1. `system_message` 直给 → 使用该内容（见 `_messages.py` 文档字符串分支说明）。
2. 否则默认拼装：`description`、`role`、`instructions`、markdown 附加段等按 `# 3.x` 注释顺序合并。

### 还原后的完整 System 文本

```text
--- instructions ---
You are a coding agent. You write clean, well-documented Python code.

## Workflow

1. Understand the task
2. Write the code and save it to a file
3. Run the file to verify it works
4. If there are errors, fix them and re-run

## Rules

- Always save code to files before running
- Include type hints on function signatures
- Add a brief docstring to each function
- Test with at least 2-3 example inputs
- No emojis
```

### 段落释义（模型视角）

- 指令与安全边界由 `instructions` / `system_message` 约束；若带 `tools` / `knowledge`，文档中需体现「何时检索/调用」由框架注入的提示段支持。

## 完整 API 请求

```python
# 请以本文件实际 Model 为准打开 libs/agno/agno/models/<厂商>/ 下对应类的 invoke：
# 可能是 chat.completions.create、responses.create、Gemini generate_content 等。
```

> 与上一节 system 文本在同一 run 中组合；`developer`/`system` 角色由适配器转换。

```mermaid
flowchart TD
    Entry["用户入口<br/>`if __name__` / main"] --> Run["【关键】Agent.run / print_response"]
    Run --> Sys["【关键】get_system_message<br/>agno/agent/_messages.py L106+"]
    Sys --> Inv["【关键】Model.invoke / 提供商 API"]
    Inv --> Out["RunOutput / 流式 chunk"]
```

**【关键】节点说明：**

- **print_response / run**：用户可见的同步入口。
- **get_system_message**：系统提示拼装核心。
- **Model.invoke**：对模型提供商的实际请求。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/_messages.py` | `get_system_message()` L106+ |
| `agno/agent/agent.py` | `Agent` 运行与 CLI 输出 |
| `agno/models/` | 各厂商 `Model.invoke` |
