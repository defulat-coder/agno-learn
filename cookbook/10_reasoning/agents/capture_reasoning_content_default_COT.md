# capture_reasoning_content_default_COT.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Capture Reasoning Content
=========================

Demonstrates how to inspect reasoning_content in streaming and non-streaming runs.
"""

from agno.agent import Agent
from agno.models.openai import OpenAIChat


# ---------------------------------------------------------------------------
# Create Helpers
# ---------------------------------------------------------------------------
def print_reasoning_content(response, label: str) -> None:
    """Print a short reasoning_content report for a run response."""
    print(f"\n--- reasoning_content from {label} ---")
    if hasattr(response, "reasoning_content") and response.reasoning_content:
        print("[OK] reasoning_content FOUND")
        print(f"   Length: {len(response.reasoning_content)} characters")
        print("\n=== reasoning_content preview ===")
        preview = response.reasoning_content[:1000]
        if len(response.reasoning_content) > 1000:
            preview += "..."
        print(preview)
    else:
        print("[NOT FOUND] reasoning_content NOT FOUND")


# ---------------------------------------------------------------------------
# Run Examples
# ---------------------------------------------------------------------------
def run_examples() -> None:
    print("\n=== Example 1: Using reasoning=True (default COT) ===\n")

    agent = Agent(
        model=OpenAIChat(id="gpt-4o"),
        reasoning=True,
        markdown=True,
    )

    print("Running with reasoning=True (non-streaming)...")
    response = agent.run("What is the sum of the first 10 natural numbers?")
    print_reasoning_content(response, label="non-streaming response")

    print("\n\n=== Example 2: Using a custom reasoning_model ===\n")

    agent_with_reasoning_model = Agent(
        model=OpenAIChat(id="gpt-4o"),
        reasoning_model=OpenAIChat(id="gpt-4o"),
        markdown=True,
    )

    print("Running with reasoning_model specified (non-streaming)...")
    response = agent_with_reasoning_model.run(
        "What is the sum of the first 10 natural numbers?"
    )
    print_reasoning_content(response, label="non-streaming response")

    print("\n\n=== Example 3: Processing stream with reasoning=True ===\n")

    streaming_agent = Agent(
        model=OpenAIChat(id="gpt-4o"),
        reasoning=True,
        markdown=True,
    )

    print("Running with reasoning=True (streaming)...")
    final_response = None
    for event in streaming_agent.run(
        "What is the value of 5! (factorial)?",
        stream=True,
        stream_events=True,
    ):
        if hasattr(event, "content") and event.content:
            print(event.content, end="", flush=True)

        if hasattr(event, "reasoning_content"):
            final_response = event

    print_reasoning_content(final_response, label="final stream event")

    print("\n\n=== Example 4: Processing stream with reasoning_model ===\n")

    streaming_agent_with_model = Agent(
        model=OpenAIChat(id="gpt-4o"),
        reasoning_model=OpenAIChat(id="gpt-4o"),
        markdown=True,
    )

    print("Running with reasoning_model specified (streaming)...")
    final_response_with_model = None
    for event in streaming_agent_with_model.run(
        "What is the value of 7! (factorial)?",
        stream=True,
        stream_events=True,
    ):
        if hasattr(event, "content") and event.content:
            print(event.content, end="", flush=True)

        if hasattr(event, "reasoning_content"):
            final_response_with_model = event

    print_reasoning_content(
        final_response_with_model,
        label="final stream event (reasoning_model)",
    )


if __name__ == "__main__":
    run_examples()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/10_reasoning/agents/capture_reasoning_content_default_COT.py`

## 概述

本示例演示如何检查 **`RunOutput.reasoning_content`**：非流式与流式、`reasoning=True`，并打印长度与预览（前 1000 字符）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `agent` | `reasoning=True`，`gpt-4o` | 默认 COT |
| 辅助函数 | `print_reasoning_content` | 调试输出 |

## 核心组件解析

用于验证推理文本是否在流/非流路径上均正确填充；具体字段由模型适配器写入 `RunOutput`。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/run/agent.py` | `RunOutput` |
| `agno/models/openai` | 推理字段解析 |
