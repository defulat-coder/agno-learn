# slack_tools.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Run: pip install openai slack-sdk"""

from agno.agent import Agent
from agno.tools.slack import SlackTools

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


# Example 1: Enable all Slack tools
agent_all = Agent(
    tools=[
        SlackTools(
            all=True,  # Enable all Slack tools
        )
    ],
    markdown=True,
)

# Example 2: Enable specific tools only
agent_specific = Agent(
    tools=[
        SlackTools(
            enable_send_message=True,
            enable_list_channels=True,
            enable_get_channel_history=False,
            enable_upload_file=False,
            enable_download_file=False,
        )
    ],
    markdown=True,
)

# Example 3: Read-only agent (no send_message)
agent_readonly = Agent(
    tools=[
        SlackTools(
            enable_send_message=False,
            enable_list_channels=True,
            enable_get_channel_history=True,
            enable_upload_file=False,
            enable_download_file=True,
        )
    ],
    markdown=True,
)

# Run examples

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    agent_all.print_response(
        "Send 'Hello from Agno!' to #general",
        stream=True,
    )

    agent_specific.print_response(
        "List all channels in the workspace",
        stream=True,
    )

    agent_readonly.print_response(
        "Get the last 5 messages from #general",
        stream=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/slack_tools.py`

## 概述

本示例展示 **`SlackTools`** 的 **`all=True`** 与细粒度 **`enable_*`**，以及只读型组合；`__main__` 对三个 Agent 调用 `print_response(..., stream=True)`。

**核心配置一览（`agent_all`）**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `tools` | `[SlackTools(all=True)]` |  |
| `markdown` | `True` | Agent 级 |

## 运行机制与因果链

需 Slack SDK 与 token；工具涵盖发消息、列频道、历史等。

## System Prompt 组装

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["enable_* 矩阵"] --> B["【关键】Slack 能力裁剪"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/slack/` | `SlackTools` |
