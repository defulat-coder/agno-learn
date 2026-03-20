# is_9_11_bigger_than_9_9.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Decimal Comparison Reasoning
============================

Demonstrates regular, built-in, and DeepSeek-backed reasoning for 9.11 vs 9.9.
"""

from agno.agent import Agent
from agno.models.anthropic import Claude
from agno.models.deepseek import DeepSeek
from agno.models.openai import OpenAIChat
from rich.console import Console

# ---------------------------------------------------------------------------
# Create Agents
# ---------------------------------------------------------------------------
console = Console()

task = "9.11 and 9.9 -- which is bigger?"

regular_agent_openai = Agent(model=OpenAIChat(id="gpt-4o"), markdown=True)

cot_agent_openai = Agent(
    model=OpenAIChat(id="gpt-4o"),
    reasoning=True,
    markdown=True,
)

regular_agent_claude = Agent(model=Claude("claude-3-5-sonnet-20241022"), markdown=True)

deepseek_agent_claude = Agent(
    model=Claude("claude-3-5-sonnet-20241022"),
    reasoning_model=DeepSeek(id="deepseek-reasoner"),
    markdown=True,
)

deepseek_agent_openai = Agent(
    model=OpenAIChat(id="gpt-4o"),
    reasoning_model=DeepSeek(id="deepseek-reasoner"),
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agents
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    console.rule("[bold blue]Regular OpenAI Agent[/bold blue]")
    regular_agent_openai.print_response(task, stream=True)

    console.rule("[bold yellow]OpenAI Built-in Reasoning Agent[/bold yellow]")
    cot_agent_openai.print_response(task, stream=True, show_full_reasoning=True)

    console.rule("[bold green]Regular Claude Agent[/bold green]")
    regular_agent_claude.print_response(task, stream=True)

    console.rule("[bold cyan]Claude + DeepSeek Reasoning Agent[/bold cyan]")
    deepseek_agent_claude.print_response(task, stream=True)

    console.rule("[bold magenta]OpenAI + DeepSeek Reasoning Agent[/bold magenta]")
    deepseek_agent_openai.print_response(task, stream=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/10_reasoning/agents/is_9_11_bigger_than_9_9.py`

## 概述

本示例对比 **无推理 / 内置推理 / Claude / DeepSeek 推理模型** 在 **9.11 vs 9.9** 小数比较上的表现：多 `Agent` 变体并列运行（`rich` 控制台输出）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `regular_agent_openai` | 仅 `gpt-4o` | 基线 |
| `cot_agent_openai` | `reasoning=True` | OpenAI COT |
| `regular_agent_claude` | `Claude` | 基线 |
| `deepseek_agent_claude` / `deepseek_agent_openai` | `reasoning_model=DeepSeek` | 外接推理 |

### 还原 task

```text
9.11 and 9.9 -- which is bigger?
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/anthropic` | Claude |
| `agno/models/deepseek` | DeepSeek |
