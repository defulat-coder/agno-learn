# text_to_speech_agent.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Example: Using the OpenAITools Toolkit for Text-to-Speech

This script demonstrates how to use an agent to generate speech from a given text input and optionally save it to a specified audio file.

Run `uv pip install openai agno` to install the necessary dependencies.
"""

import base64
from pathlib import Path

from agno.agent import Agent
from agno.models.google import Gemini
from agno.tools.openai import OpenAITools
from agno.utils.media import save_base64_data

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------

output_file: str = str(Path("tmp/speech_output.mp3"))

agent: Agent = Agent(
    model=Gemini(id="gemini-2.5-pro"),
    tools=[OpenAITools(enable_speech_generation=True)],
    markdown=True,
)

# Ask the agent to generate speech, but not save it
response = agent.run(
    'Please generate speech for the following text: "Hello from Agno! This is a demonstration of the text-to-speech capability using OpenAI"'
)

print(f"Agent response: {response.get_content_as_string()}")

if response.audio:
    base64_audio = base64.b64encode(response.audio[0].content).decode("utf-8")
    save_base64_data(base64_audio, output_file)
    print(f"Successfully saved generated speech to{output_file}")

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    pass
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/90_models/openai/chat/text_to_speech_agent.py`

## 概述

本文件位于 `openai/chat/` 目录，但 **模型为 `Gemini(id="gemini-2.5-pro")` + `OpenAITools(enable_speech_generation=True)`**，演示通过 OpenAI 工具包做 **TTS**，而非 `OpenAIChat` 直连。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-2.5-pro")` | 非 OpenAIChat |
| `tools` | `[OpenAITools(enable_speech_generation=True)]` | 语音生成 |
| `markdown` | `True` | 默认 |

### 与「单一 OpenAI Agent」示例的差异

不存在以 `OpenAIChat` 为主的单一链路；**system 拼装仍走 `Agent` + `Gemini` 适配器**，须读 `agno/models/google` 的 `get_system_message` 等价逻辑（若与 OpenAI 分叉则在对应模型消息层）。

用户消息（`agent.run` 输入）：请朗读固定英文句子（见源码字符串）。

## Mermaid 流程图

```mermaid
flowchart TD
    A["Gemini Agent"] --> B["【关键】OpenAITools 语音生成"]
    B --> C["response.audio 写文件"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/openai.py` | `OpenAITools` |
| `agno/models/google/` | `Gemini` |
