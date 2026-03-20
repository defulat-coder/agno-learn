# finance_agent.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Reasoning Finance Agent
=======================

Demonstrates built-in and DeepSeek-backed reasoning for financial reporting.
"""

from agno.agent import Agent
from agno.models.deepseek import DeepSeek
from agno.models.openai import OpenAIChat
from agno.tools.yfinance import YFinanceTools

# ---------------------------------------------------------------------------
# Create Agents
# ---------------------------------------------------------------------------
cot_agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[YFinanceTools()],
    instructions="Use tables to display data",
    use_json_mode=True,
    reasoning=True,
    markdown=True,
)

deepseek_agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[YFinanceTools()],
    instructions=["Use tables where possible"],
    reasoning_model=DeepSeek(id="deepseek-reasoner"),
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agents
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    prompt = "Write a report comparing NVDA to TSLA"

    print("=== Built-in Chain Of Thought ===")
    cot_agent.print_response(prompt, stream=True, show_full_reasoning=True)

    print("\n=== DeepSeek Reasoning Model ===")
    deepseek_agent.print_response(prompt, stream=True, show_full_reasoning=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/10_reasoning/agents/finance_agent.py`

## 概述

本示例在 **YFinance 工具** 上对比 **`reasoning=True`** 与 **`reasoning_model=DeepSeek`**；`cot_agent` 使用 `use_json_mode=True` 与 `instructions` 表格展示。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `tools` | `[YFinanceTools()]` | 行情数据 |
| `cot_agent.instructions` | `"Use tables to display data"` | 单字符串 |
| `deepseek_agent.instructions` | `["Use tables where possible"]` | 列表 |

### 还原 instructions

```text
Use tables to display data
```

```text
Use tables where possible
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/yfinance` | 金融工具 |
