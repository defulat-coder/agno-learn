# 02_agentic_learn.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Learning Machines: Agentic Mode
===============================
In AGENTIC mode, the agent receives tools to explicitly manage learning.
It decides when to save profiles and memories based on conversation context.

Compare with learning=True (ALWAYS mode) where extraction happens automatically.
"""

from agno.agent import Agent
from agno.db.sqlite import SqliteDb
from agno.learn import (
    LearningMachine,
    LearningMode,
    UserMemoryConfig,
    UserProfileConfig,
)
from agno.models.openai import OpenAIResponses

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
db = SqliteDb(db_file="tmp/agents.db")

agent = Agent(
    model=OpenAIResponses(id="gpt-5.2"),
    db=db,
    learning=LearningMachine(
        user_profile=UserProfileConfig(mode=LearningMode.AGENTIC),
        user_memory=UserMemoryConfig(mode=LearningMode.AGENTIC),
    ),
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Demo
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    user_id = "alice2@example.com"

    # Session 1: Agent decides what to save via tool calls
    print("\n--- Session 1: Agent uses tools to save profile and memories ---\n")
    agent.print_response(
        "Hi! I'm Alice. I work at Anthropic as a research scientist. "
        "I prefer concise responses without too much explanation.",
        user_id=user_id,
        session_id="session_1",
        stream=True,
    )
    lm = agent.learning_machine
    lm.user_profile_store.print(user_id=user_id)
    lm.user_memory_store.print(user_id=user_id)

    # Session 2: New session - agent remembers
    print("\n--- Session 2: Agent remembers across sessions ---\n")
    agent.print_response(
        "What do you know about me?",
        user_id=user_id,
        session_id="session_2",
        stream=True,
    )
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/08_learning/00_quickstart/02_agentic_learn.py`

## 概述

本示例展示 Agno 的 **`LearningMachine` + `LearningMode.AGENTIC`** 机制：为用户画像与用户记忆显式注册工具，由模型在对话中决定何时写入/更新，便于观察工具调用与可控写入。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(id="gpt-5.2")` | OpenAI Responses API |
| `db` | `SqliteDb(db_file="tmp/agents.db")` | 持久化 |
| `learning` | `LearningMachine(user_profile=UserProfileConfig(mode=AGENTIC), user_memory=UserMemoryConfig(mode=AGENTIC))` | 画像与记忆均为 AGENTIC |
| `markdown` | `True` | Markdown 格式化提示 |
| `name` / `description` / `instructions` | 未设置 | 未设置 |

## 架构分层

```
用户代码层                agno.agent 层
┌──────────────────┐    ┌──────────────────────────────────┐
│ LearningMachine  │    │ Agent：工具列表含 profile/memory   │
│ AGENTIC x2       │───>│  get_system_message + 工具说明     │
│                  │    │  invoke：模型可发 tool calls       │
└──────────────────┘    └──────────────────────────────────┘
                                │
                                ▼
                        ┌──────────────────┐
                        │ OpenAIResponses  │
                        └──────────────────┘
```

## 核心组件解析

### AGENTIC 与工具暴露

`UserProfileConfig(mode=AGENTIC)` 与 `UserMemoryConfig(mode=AGENTIC)` 会使对应 store 在 system 中注入工具说明文档（见 `UserProfileStore.build_context` / `UserMemoryStore.build_context` 中 `_should_expose_tools` 分支），并在 `get_tools()` 中注册可调用工具，模型通过 Responses API 的 `tools` 参数发起调用。

### 运行机制与因果链

1. **数据路径**：用户自然语言 → 模型可能先 `update_user_profile` / `update_user_memory`（名称以实际注册为准）→ 再生成对用户可见的文本回复。
2. **副作用**：写入 SQLite；`learning_machine.*_store.print` 用于调试查看。
3. **分支**：同目录 `01_always_learn.py` 为并行抽取；本文件强调**可观测工具调用**。
4. **定位**：快速对比 ALWAYS vs AGENTIC 的双模式入门。

## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 1 | `instructions` | 无 | 否 |
| 2 | `markdown` | 追加「Use markdown…」 | 是 |
| 3 | 工具与学习说明 | AGENTIC store 注入的工具文档 + `_learning.build_context` | 是 |
| 4 | Model 层 | `get_system_message_for_model` | 视模型 |

### 拼装顺序与源码锚点

在 `get_system_message()` 中：`# 3.3.3` 指令 → `# 3.3.4` additional_information（含 markdown）→ `# 3.3.5` `_tool_instructions`（若有）→ … → `# 3.3.12` 学习上下文。工具 schema 由 Agent 工具管线挂载，与 `_messages.py` 中系统文案相互配合。

### 还原后的完整 System 文本

本文件未提供自定义 `instructions`/`description`。可静态确定的部分包含：

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

AGENTIC 模式下，`<user_profile>` / `<user_memory>` 内会附带**工具用法说明**（源码见 `agno/learn/stores/user_profile.py`、`user_memory.py` 中 `build_context`）；完整正文随是否已有数据、以及模型 ID 而变化，需运行时打印 `get_system_message` 返回值核对。

### 段落释义（模型视角）

- 工具段约束「何时更新画像/记忆」，与 ALWAYS 的隐式抽取形成对照。
- Markdown 段约束输出形态。

### 与 User 消息的边界

用户消息仅为自然语言请求；是否写入记忆由模型对工具调用的决策决定。

## 完整 API 请求

```python
# agno/models/openai/responses.py — OpenAIResponses.invoke
client.responses.create(
    model="gpt-5.2",
    input=[...],  # 含 developer/system 与 user 内容，由 _format_messages 生成
    tools=[...],  # AGENTIC 学习工具
)
```

## Mermaid 流程图

```mermaid
flowchart TD
    U["用户输入"] --> R["Agent._run"]
    R --> S["【关键】工具 + learning system 提示"]
    S --> M["responses.create"]
    M --> T{{ "tool_calls?" }}
    T -->|是| L["学习 store 落库"]
    T -->|否| O["直接文本回复"]
    L --> O
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/learn/machine.py` | `LearningMachine` | 聚合各 store |
| `agno/learn/stores/user_profile.py` | `build_context` / 工具文档 | AGENTIC 说明与上下文 |
| `agno/learn/stores/user_memory.py` | `build_context` | 同上 |
| `agno/agent/_messages.py` | `get_system_message` L399-407 | #3.3.12 学习块 |
