# 02_session_state_tools.py — 实现原理分析

> 源文件：`cookbook/02_agents/04_tools/02_session_state_tools.py`

## 概述

本示例展示 Agno 的 **`session_state` 直接注入 + `cache_callables=False`** 机制：工厂函数的参数名声明为 `session_state`（而非 `run_context`），框架会直接注入 session state 字典。同时设置 `cache_callables=False`，确保每次运行都重新调用工厂，实时响应 session_state 的变化。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(id="gpt-5-mini")` | Responses API |
| `tools` | `get_tools`（callable） | 可调用工具工厂函数 |
| `cache_callables` | `False` | 禁用缓存，每次运行重新调用工厂 |
| `instructions` | `["Use the available tool to respond."]` | 单条指令 |
| `name` | `None` | 未设置 |
| `description` | `None` | 未设置 |
| `markdown` | `False`（默认） | 未设置 |

## 架构分层

```
用户代码层                          agno 内部层
┌──────────────────────────┐      ┌────────────────────────────────────────┐
│ 02_session_state_tools.py│      │ Agent._run()                           │
│                          │      │  ├─ _tools.get_tools()                 │
│ tools=get_tools          │─────>│  │  ├─ resolve_callable_tools()        │
│   (callable factory)     │      │  │  │  ├─ cache_callables=False        │
│                          │      │  │  │  │  → 跳过缓存检查               │
│ cache_callables=False    │      │  │  │  ├─ invoke_callable_factory()    │
│                          │      │  │  │  │  └─ 注入 session_state dict   │
│ session_state=           │      │  │  │  └─ run_context.tools = result   │
│   {"mode": "greet"}      │      │  │  └─ determine_tools_for_model()     │
│                          │      │  │                                      │
│                          │      │  ├─ _messages.get_run_messages()       │
└──────────────────────────┘      └────────────────────────────────────────┘
                                           │
                                           ▼
                                   ┌───────────────────┐
                                   │ OpenAIResponses    │
                                   │ gpt-5-mini         │
                                   └───────────────────┘
```

## 核心组件解析

### session_state 参数注入

与 `01_callable_tools.py` 不同，本文件的工厂函数直接声明 `session_state: dict`：

```python
# 02_session_state_tools.py:L29-37
def get_tools(session_state: dict):
    mode = session_state.get("mode", "greet")
    if mode == "greet":
        return [get_greeting]
    else:
        return [get_farewell]
```

框架在 `invoke_callable_factory()`（`utils/callables.py:L60`）中通过签名检查注入：

```python
# utils/callables.py:L88-89
if "session_state" in sig.parameters:
    kwargs["session_state"] = run_context.session_state if run_context.session_state is not None else {}
```

两种注入方式对比：

| 参数名 | 注入内容 | 适用场景 |
|--------|---------|---------|
| `run_context` | 完整 RunContext 对象（含 user_id, session_id, session_state 等） | 需要多种上下文信息 |
| `session_state` | 仅 session_state 字典 | 只需要读取状态值 |

### cache_callables=False

`cache_callables=False` 的效果在 `resolve_callable_tools()`（`utils/callables.py:L213`）中体现：

```python
# utils/callables.py:L222-248
cache_enabled = getattr(entity, "cache_callables", True)  # 本文件为 False

# 由于 cache_enabled=False，以下条件永远不满足
if cache_enabled and cache_key is not None and cache_key in cache:
    ...  # 跳过

# 工厂每次都被调用
result = invoke_callable_factory(entity.tools, entity, run_context)

# 也不缓存结果
if cache_enabled and cache_key is not None:  # False，跳过
    cache[cache_key] = result
```

| 配置 | 行为 | 适用场景 |
|------|------|---------|
| `cache_callables=True`（默认） | 按 user_id/session_id 缓存 | 角色固定，同一用户工具不变 |
| `cache_callables=False` | 每次运行重新调用 | session_state 频繁变化 |

### 运行时工具切换

```
Run 1: session_state={"mode": "greet"}
  → get_tools() 返回 [get_greeting]
  → 模型只能调用 get_greeting

Run 2: session_state={"mode": "farewell"}
  → get_tools() 重新调用（无缓存）
  → 返回 [get_farewell]
  → 模型只能调用 get_farewell
```

## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 1 | `system_message`（自定义） | `None` | 否 |
| 3.1 | `instructions` | `["Use the available tool to respond."]` | 是 |
| 3.1.1 | 模型指令（`get_instructions_for_model`） | 取决于模型 | 是 |
| 3.2.1 | `markdown` | `False` | 否 |
| 3.2.2 | `add_datetime_to_context` | `False` | 否 |
| 3.2.3 | `add_location_to_context` | `False` | 否 |
| 3.2.4 | `add_name_to_context` | `False` | 否 |
| 3.3.1 | `description` | `None` | 否 |
| 3.3.2 | `role` | `None` | 否 |
| 3.3.3 | instructions 拼接 | 单条指令 | 是 |
| 3.3.4 | additional_information | 无 | 否 |
| 3.3.5 | `_tool_instructions` | 无 | 否 |
| 3.3.7 | `expected_output` | `None` | 否 |
| 3.3.8 | `additional_context` | `None` | 否 |
| 3.3.9 | `add_memories_to_context` | `None` | 否 |

### 最终 System Prompt

```text
Use the available tool to respond.
```

## 完整 API 请求

**Run 1（greet 模式）：**

```python
client.responses.create(
    model="gpt-5-mini",
    input=[
        {"role": "developer", "content": "Use the available tool to respond.\n\n"},
        {"role": "user", "content": "Say hi to Alice"}
    ],
    tools=[
        {
            "type": "function",
            "name": "get_greeting",
            "description": "Greet someone by name.",
            "parameters": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"}
                },
                "required": ["name"]
            }
        }
    ],
    stream=True,
    stream_options={"include_usage": True}
)
```

**Run 2（farewell 模式）：**

```python
client.responses.create(
    model="gpt-5-mini",
    input=[
        {"role": "developer", "content": "Use the available tool to respond.\n\n"},
        {"role": "user", "content": "Say bye to Alice"}
    ],
    tools=[
        {
            "type": "function",
            "name": "get_farewell",
            "description": "Say goodbye to someone.",
            "parameters": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"}
                },
                "required": ["name"]
            }
        }
    ],
    stream=True,
    stream_options={"include_usage": True}
)
```

> **关键区别**：两次运行的 `tools` 数组完全不同，模型在每次运行中只能看到当前模式对应的工具。

## Mermaid 流程图

```mermaid
flowchart TD
    A["用户代码<br/>agent.print_response(...)"] --> B["Agent._run()"]
    B --> C["_tools.get_tools()"]

    subgraph 工具解析（每次运行）
        C --> C1["resolve_callable_tools()"]
        C1 --> C2{"cache_callables?"}
        C2 -->|False| C3["invoke_callable_factory()<br/>签名注入 session_state"]
        C3 --> C4["get_tools(session_state)<br/>根据 mode 返回工具"]
        C4 --> C5["run_context.tools = result<br/>不缓存"]
    end

    C5 --> D["determine_tools_for_model()"]
    D --> E["get_run_messages()"]
    E --> F["get_system_message()"]
    F --> G["OpenAIResponses.invoke()"]
    G --> H["模型响应"]
    H --> I{"有工具调用?"}
    I -->|是| J["执行工具函数"]
    J --> G
    I -->|否| K["流式输出"]

    style A fill:#e1f5fe
    style K fill:#e8f5e9
    style C3 fill:#fff3e0
    style C4 fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `tools` L159 | 工具参数定义，支持 `Callable` |
| `agno/agent/agent.py` | `cache_callables` L352 | 控制是否缓存工厂结果 |
| `agno/utils/callables.py` | `resolve_callable_tools()` L213 | 解析 callable 工具工厂 |
| `agno/utils/callables.py` | `invoke_callable_factory()` L60-101 | 签名检查 + session_state 注入 |
| `agno/utils/callables.py` | `_compute_cache_key()` L133 | 缓存 key 计算（cache_callables=False 时不生效） |
| `agno/agent/_tools.py` | `get_tools()` L105 | 工具收集入口 |
| `agno/agent/_tools.py` | `determine_tools_for_model()` L434 | 转换为模型可用的 Function 列表 |
| `agno/agent/_messages.py` | `get_system_message()` L106 | 构造 system prompt |
