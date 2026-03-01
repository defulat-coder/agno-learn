# stream_hook.py — 实现原理分析

> 源文件：`cookbook/02_agents/09_hooks/stream_hook.py`

## 概述

本示例展示 Agno 的 **`post_hooks` + `metadata`** 联动机制：在模型生成响应后，通过 post_hook 访问 `RunContext.metadata` 中的用户元数据（如邮箱），将响应内容发送通知（模拟发送邮件）。展示了 hook 函数如何利用 `run_output` 和 `run_context` 两个参数实现副作用操作。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Financial Report Agent"` | Agent 名称 |
| `model` | `OpenAIResponses(id="gpt-5-mini")` | Responses API |
| `post_hooks` | `[send_notification]` | 后置 hook（发送通知） |
| `pre_hooks` | `None` | 未设置 |
| `tools` | `[YFinanceTools()]` | Yahoo Finance 工具 |
| `instructions` | 3 条金融报告指令 | 专业领域指令 |
| `markdown` | `False` | 默认值 |

> 本示例使用异步模式（`aprint_response` + `asyncio.run`），并传递 `stream=True` 和 `metadata={"email": "test@example.com"}`。

## 架构分层

```
用户代码层                          agno.agent 层
┌──────────────────────────┐    ┌──────────────────────────────────────┐
│ stream_hook.py           │    │ Agent._run()                         │
│                          │    │  ├ normalize_post_hooks()             │
│ post_hooks=[             │    │  │    → 普通函数直接保留              │
│   send_notification      │───>│  │                                    │
│ ]                        │    │  ├ get_tools() → YFinanceTools        │
│                          │    │  ├ get_run_messages()                 │
│ tools=[YFinanceTools()]  │    │  ├ Model.response()                   │
│                          │    │  │    → 可能包含工具调用循环          │
│ metadata={               │    │  │                                    │
│   "email": "test@..."    │    │  ├ execute_post_hooks()               │
│ }                        │    │  │    → filter_hook_args()            │
│                          │    │  │    → hook(run_output, run_context)  │
│ stream=True              │    │  │    → run_context.metadata["email"] │
│ user_id="user_123"       │    │  │    → send_email(email, content)    │
└──────────────────────────┘    └──────────────────────────────────────┘
                                        │
                                        ▼
                                ┌──────────────────┐
                                │ OpenAIResponses   │
                                │ gpt-5-mini        │
                                └──────────────────┘
```

## 核心组件解析

### post_hook 双参数注入：run_output + run_context

`send_notification` 函数签名同时接受 `run_output` 和 `run_context`：

```python
def send_notification(run_output: RunOutput, run_context: RunContext) -> None:
    if run_context.metadata is None:
        return
    email = run_context.metadata.get("email")
    if email:
        send_email(email, run_output.content)
```

`filter_hook_args()`（`utils/hooks.py:156`）从 `all_args` 中匹配签名参数。post_hooks 的 `all_args` 中同时包含 `run_output` 和 `run_context`：

```python
# _hooks.py L284-293
all_args = {
    "run_output": run_output,      # 模型响应（含 content）
    "run_context": run_context,    # 运行上下文（含 metadata）
    "agent": agent,
    "session": session,
    "user_id": user_id,
    ...
}
```

### metadata 传递链

`metadata` 通过 `aprint_response()` 传入，经过以下路径到达 hook 函数：

```
aprint_response(metadata={"email": "test@example.com"})
  → _run() 构建 RunContext(metadata={"email": "test@example.com"})
    → execute_post_hooks() 中 all_args["metadata"] = run_context.metadata
    → filter_hook_args() 筛选 run_context 参数
      → hook 访问 run_context.metadata.get("email")
```

### 非阻断式 post_hook

与前两个示例（`validate_response_quality` / `simple_length_validation`）不同，`send_notification` **不抛出 `OutputCheckError`**——它执行副作用操作（发送通知）但不阻断响应返回。如果 hook 执行过程中出现异常，仅记录日志而不影响正常响应：

```python
# _hooks.py L361-363
except Exception as e:
    log_error(f"Post-hook #{i + 1} execution failed: {str(e)}")
    log_exception(e)  # 记录日志，不中断
```

### YFinanceTools 工具集成

Agent 配置了 `tools=[YFinanceTools()]`，模型可能在生成金融报告时调用 Yahoo Finance API 获取实时数据。工具调用完成后才执行 post_hook。

## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 1 | `system_message`（自定义） | `None` | 否 |
| 3.1 | `instructions` | 3 条金融报告指令 | 是 |
| 3.1.1 | 模型指令（`get_instructions_for_model`） | 模型默认 | 是 |
| 3.2.1 | `markdown` | `False` | 否 |
| 3.2.2 | `add_datetime_to_context` | `False` | 否 |
| 3.2.3 | `add_location_to_context` | `False` | 否 |
| 3.2.4 | `add_name_to_context` | `False` | 否 |
| 3.3.1 | `description` | `None` | 否 |
| 3.3.2 | `role` | `None` | 否 |
| 3.3.3 | instructions 拼接 | 3 条指令拼接 | 是 |
| 3.3.4 | additional_information | 无 | 否 |
| 3.3.5 | `_tool_instructions` | YFinanceTools 工具使用说明 | 是 |
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
- You are a helpful financial report agent.
- Generate a financial report for the given company.
- Keep it short and concise.

<YFinanceTools 工具使用说明>
```

## 完整 API 请求

### 第一轮：模型调用（可能触发工具调用）

```python
client.responses.create(
    model="gpt-5-mini",
    input=[
        # System Message
        {"role": "developer", "content": "- You are a helpful financial report agent.\n- Generate a financial report...\n- Keep it short and concise.\n\n<tool_instructions>..."},
        # 用户输入
        {"role": "user", "content": "Generate a financial report for Apple (AAPL)."}
    ],
    tools=[
        # YFinanceTools 提供的工具（如 get_stock_price, get_company_info 等）
        {"type": "function", "function": {"name": "get_stock_price", "parameters": {...}}},
        {"type": "function", "function": {"name": "get_company_info", "parameters": {...}}},
        # ...
    ],
    stream=True,
    stream_options={"include_usage": True}
)
```

### 第二轮：工具结果回传后

```python
client.responses.create(
    model="gpt-5-mini",
    input=[
        {"role": "developer", "content": "..."},
        {"role": "user", "content": "Generate a financial report for Apple (AAPL)."},
        # 模型发起工具调用
        {"type": "function_call", "name": "get_stock_price", "arguments": "{\"symbol\": \"AAPL\"}"},
        # 工具执行结果
        {"type": "function_call_output", "call_id": "...", "output": "{\"price\": 185.50, ...}"}
    ],
    tools=[...],
    stream=True,
    stream_options={"include_usage": True}
)
```

### post_hook 执行（模型响应完成后）

```python
# 不产生 API 请求
# send_notification() 读取 run_output.content 和 run_context.metadata
# 模拟发送邮件：print(f"Sending email to test@example.com: <报告内容>")
```

> post_hook 的执行不消耗额外 token，仅执行本地副作用操作。

## Mermaid 流程图

```mermaid
flowchart TD
    A["用户代码<br/>agent.aprint_response(<br/>metadata={'email': 'test@example.com'},<br/>stream=True)"] --> B["Agent._run()"]
    B --> C["get_tools()<br/>YFinanceTools"]
    C --> D["get_run_messages()"]
    D --> E["OpenAIResponses.invoke()<br/>gpt-5-mini"]

    E --> F{"需要工具调用?"}
    F -->|"是"| G["调用 YFinance API"]
    G --> H["工具结果回传"]
    H --> E
    F -->|"否"| I["模型生成最终响应"]

    I --> J["execute_post_hooks()"]

    subgraph Post-Hook：发送通知
        J --> K["filter_hook_args()<br/>注入 run_output + run_context"]
        K --> L["send_notification()"]
        L --> M{"metadata 有 email?"}
        M -->|"是"| N["send_email()<br/>发送通知"]
        M -->|"否"| O["跳过"]
    end

    N --> P["返回流式响应"]
    O --> P

    style A fill:#e1f5fe
    style P fill:#e8f5e9
    style L fill:#fff3e0
    style N fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `post_hooks` L178 | post_hooks 属性定义 |
| `agno/agent/_run.py` | `_run()` L559-572 | 步骤 10：执行 post_hooks |
| `agno/agent/_hooks.py` | `execute_post_hooks()` L266 | 同步版 post_hooks 执行入口 |
| `agno/agent/_hooks.py` | `aexecute_post_hooks()` L369 | 异步版 post_hooks 执行入口 |
| `agno/agent/_hooks.py` | L284-293 | 构建 all_args（含 run_output + metadata） |
| `agno/utils/hooks.py` | `filter_hook_args()` L156 | 按签名过滤参数 |
| `agno/run/base.py` | `RunContext` L16 | 运行上下文（含 metadata） |
| `agno/run/agent.py` | `RunOutput` L581 | 模型输出容器（含 content） |
