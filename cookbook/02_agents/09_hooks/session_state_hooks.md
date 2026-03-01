# session_state_hooks.py — 实现原理分析

> 源文件：`cookbook/02_agents/09_hooks/session_state_hooks.py`

## 概述

本示例展示 Agno 的 **`pre_hooks` + `session_state`** 联动机制：通过 pre_hook 函数在每次运行前用独立 Agent 分析用户输入并提取话题关键词，将结果累积写入 `RunContext.session_state`，借助 `SqliteDb` 实现跨运行的状态持久化。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Simple Agent"` | Agent 名称 |
| `model` | `OpenAIResponses(id="gpt-5-mini")` | Responses API |
| `pre_hooks` | `[track_conversation_topics]` | 前置 hook（话题追踪函数） |
| `post_hooks` | `None` | 未设置 |
| `db` | `SqliteDb(db_file="test.db")` | SQLite 持久化 |
| `session_state` | `None`（hook 内动态初始化） | 通过 RunContext 动态管理 |
| `add_session_state_to_context` | `False` | 未注入 system prompt |
| `enable_agentic_state` | `False` | 未启用内置状态工具 |
| `instructions` | `None` | 未设置 |
| `tools` | `None` | 未设置 |

## 架构分层

```
用户代码层                          agno.agent 层
┌──────────────────────────┐    ┌──────────────────────────────────────┐
│ session_state_hooks.py   │    │ Agent._run()                         │
│                          │    │  ├ _run.py L1250-1256                │
│ pre_hooks=[              │    │  │  normalize_pre_hooks()             │
│   track_conversation_    │───>│  │                                    │
│   topics                 │    │  ├ _run.py L410-425                  │
│ ]                        │    │  │  execute_pre_hooks()               │
│                          │    │  │    → filter_hook_args()            │
│ db=SqliteDb("test.db")   │    │  │    → hook(run_context, run_input)  │
│                          │    │  │    → run_context.session_state      │
│ session_id=              │    │  │      ["topics"].extend(...)        │
│   "topics_analyzer_      │    │  │                                    │
│    session"              │    │  ├ get_run_messages()                 │
│                          │    │  ├ Model.response()                   │
│                          │    │  └ cleanup_and_store()                │
│                          │    │      → 持久化 session_state 到 DB     │
└──────────────────────────┘    └──────────────────────────────────────┘
                                        │
                                        ▼
                                ┌──────────────────┐
                                │ OpenAIResponses   │
                                │ gpt-5-mini        │
                                └──────────────────┘
```

## 核心组件解析

### pre_hook 参数注入：run_context + run_input

`track_conversation_topics` 函数签名接受两个参数：

```python
def track_conversation_topics(run_context: RunContext, run_input: RunInput) -> None:
```

`filter_hook_args()`（`utils/hooks.py:156`）根据签名从 `all_args` 中筛选出 `run_context` 和 `run_input` 两个参数注入。

### RunContext.session_state 引用传递

`RunContext.session_state`（`run/base.py:27`）是字典的**引用传递**，hook 函数对其的修改直接反映到 Agent 的运行上下文中：

```python
# session_state_hooks.py L27-30
# 初始化 session_state（首次运行时）
if run_context.session_state is None:
    run_context.session_state = {"topics": []}
elif run_context.session_state.get("topics") is None:
    run_context.session_state["topics"] = []
```

hook 函数通过独立 Agent 提取话题，然后直接追加到 `session_state`：

```python
# session_state_hooks.py L56
run_context.session_state["topics"].extend(response.content.topics)
```

### session_state 持久化流程

`session_state` 的持久化通过 `cleanup_and_store()`（`_run.py:4348`）在运行结束时自动完成，将 `run_context.session_state` 写入 `SqliteDb`：

1. hook 修改 `run_context.session_state` → 运行时立即生效
2. 模型调用完成 → `cleanup_and_store()` 将 session_state 存入 `test.db`
3. 下次运行同一 `session_id` → 从 DB 加载已有 session_state
4. hook 再次运行 → 在已有 topics 上 `.extend()` 新话题

### 跨运行状态累积

示例通过指定相同的 `session_id="topics_analyzer_session"` 实现跨运行累积：

```python
# 第一次运行："I want to know more about AI Agents."
# session_state = {"topics": ["AI Agents"]}

# 第二次运行："I also want to know more about Agno..."
# session_state = {"topics": ["AI Agents", "Agno", "AI Agents"]}  # 累积
```

### get_session_state() 读取

运行后通过 `agent.get_session_state(session_id=...)`（`agent.py:939`）读取当前 session_state，该方法从内存缓存或 DB 中获取最新状态。

## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 1 | `system_message`（自定义） | `None` | 否 |
| 3.1 | `instructions` | `None` | 否 |
| 3.1.1 | 模型指令（`get_instructions_for_model`） | 模型默认 | 是 |
| 3.2.1 | `markdown` | `False` | 否 |
| 3.2.2 | `add_datetime_to_context` | `False` | 否 |
| 3.2.3 | `add_location_to_context` | `False` | 否 |
| 3.2.4 | `add_name_to_context` | `False` | 否 |
| 3.3.1 | `description` | `None` | 否 |
| 3.3.2 | `role` | `None` | 否 |
| 3.3.3 | instructions 拼接 | 无 | 否 |
| 3.3.4 | additional_information | 无 | 否 |
| 3.3.5 | `_tool_instructions` | `None` | 否 |
| fmt | `resolve_in_context` 变量替换 | 无模板变量 | 否 |
| 3.3.7 | `expected_output` | `None` | 否 |
| 3.3.8 | `additional_context` | `None` | 否 |
| 3.3.9 | `add_memories_to_context` | `None` | 否 |
| 3.3.10 | `add_culture_to_context` | `None` | 否 |
| 3.3.11 | `add_session_summary_to_context` | `None` | 否 |
| 3.3.12 | `add_learnings_to_context` | `True`（默认），但无 learning | 否 |
| 3.3.13 | `search_knowledge` instructions | 无 knowledge | 否 |
| 3.3.14 | 模型 system message | 模型默认 | 否 |
| 3.3.15 | JSON output prompt | 无 output_schema | 否 |
| 3.3.16 | response model format prompt | 无 parser_model | 否 |
| 3.3.17 | `add_session_state_to_context` | `False` | 否 |

### 最终 System Prompt

```text
（仅包含模型默认指令，无用户自定义内容）
```

> 注意：虽然 `session_state` 被修改，但 `add_session_state_to_context=False`（默认），因此 session_state 不会出现在 system prompt 中。它仅作为元数据持久化存储。

## 完整 API 请求

### 第一次运行

pre_hook 中的话题分析 Agent 请求（内部）：

```python
client.responses.create(
    model="gpt-5-mini",
    input=[
        {"role": "developer", "content": "Your task is to analyze a user query and extract the topics..."},
        {"role": "user", "content": "Extract the topics present in the following user message: I want to know more about AI Agents."}
    ],
    text={"format": {"type": "json_schema", "name": "ConversationTopics", "schema": {...}}},
    stream=True,
    stream_options={"include_usage": True}
)
```

主 Agent 请求：

```python
client.responses.create(
    model="gpt-5-mini",
    input=[
        # System Message（仅模型默认指令）
        {"role": "developer", "content": "..."},
        # 用户输入
        {"role": "user", "content": "I want to know more about AI Agents."}
    ],
    stream=True,
    stream_options={"include_usage": True}
)
```

### 第二次运行

同上，但 session_state 已从 DB 加载 `{"topics": ["AI Agents"]}`，hook 函数会在其上追加新话题。

> 注意：`session_state` 的变化不影响 API 请求格式（因为 `add_session_state_to_context=False`），它仅通过 `RunContext` 在 hook 函数和后续运行之间传递。

## Mermaid 流程图

```mermaid
flowchart TD
    A["用户代码<br/>agent.print_response(input=...,<br/>session_id='topics_analyzer_session')"] --> B["Agent._run()"]
    B --> B1["read_or_create_session()<br/>从 SqliteDb 加载 session_state"]
    B1 --> C["execute_pre_hooks()"]

    subgraph Pre-Hook：话题追踪
        C --> D["filter_hook_args()<br/>注入 run_context + run_input"]
        D --> E["track_conversation_topics()"]
        E --> F["初始化 session_state<br/>如果为 None"]
        F --> G["话题分析 Agent.run()<br/>output_schema=ConversationTopics"]
        G --> H["run_context.session_state<br/>['topics'].extend(...)"]
    end

    H --> I["get_run_messages()<br/>构建消息"]
    I --> J["OpenAIResponses.invoke()<br/>gpt-5-mini"]
    J --> K["模型响应"]
    K --> L["cleanup_and_store()<br/>session_state → SqliteDb"]
    L --> M["agent.get_session_state()<br/>读取累积话题"]

    style A fill:#e1f5fe
    style M fill:#e8f5e9
    style E fill:#fff3e0
    style H fill:#fff3e0
    style L fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `pre_hooks` L176 | pre_hooks 属性定义 |
| `agno/agent/agent.py` | `session_state` L84 | session_state 属性定义 |
| `agno/agent/agent.py` | `get_session_state()` L939 | 获取当前 session_state |
| `agno/agent/_run.py` | `_run()` L410-425 | 步骤 4：执行 pre_hooks |
| `agno/agent/_run.py` | `cleanup_and_store()` L4348 | 运行后持久化 session_state |
| `agno/agent/_hooks.py` | `execute_pre_hooks()` L43 | pre_hooks 执行入口 |
| `agno/agent/_hooks.py` | L62-70 | 构建 all_args（含 run_context） |
| `agno/utils/hooks.py` | `filter_hook_args()` L156 | 按签名过滤参数 |
| `agno/run/base.py` | `RunContext` L16 | 运行上下文（session_state 引用传递） |
| `agno/run/agent.py` | `RunInput` L29 | 用户输入容器 |
| `agno/db/sqlite/` | `SqliteDb` | SQLite 持久化后端 |
