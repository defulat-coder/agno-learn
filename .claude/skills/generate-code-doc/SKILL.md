---
name: generate-code-doc
description: Use when analyzing agno-cookbook Python source files to generate implementation principle documentation (.md). Use when the user asks to generate code analysis, implementation docs, or principle documentation for .py files.
---

# Agno 代码原理文档生成

分析 agno-cookbook 中的 Python 源文件，生成结构化的实现原理分析文档（.md），覆盖 Agent 配置、prompt 组装、API 请求、源码索引等。

## 适用场景

- 用户要求对某个 `.py` 文件生成原理分析文档
- 用户要求对某个目录批量生成所有 `.py` 文件的原理文档
- 用户提到"实现原理"、"代码分析"、"原理文档"等关键词

## 输出规范

- 文件名：`<源文件名>.md`（如 `few_shot_learning.py` → `few_shot_learning.md`）
- 输出位置：与源 `.py` 文件同目录
- 语言：中文

## 文档模板结构

每个 `.md` 文件必须包含以下章节，按顺序排列：

### 1. 标题与源文件引用

```markdown
# <文件名>.py — 实现原理分析

> 源文件：`<相对路径>/<文件名>.py`
```

### 2. 概述

一段话总结该示例展示的 Agno 核心机制，加粗关键特性名。末尾附「核心配置一览」表格：

```markdown
## 概述

本示例展示 Agno 的 **`<核心特性名>`** 机制：<一句话描述原理和用途>。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"xxx"` | Agent 名称 |
| `model` | `OpenAIChat(id="gpt-4o")` | Chat Completions API |
| `instructions` | ... | ... |
| `tools` | ... | ... |
| ... | ... | ... |
```

配置表规则：
- 列出 Agent 构造函数的**所有显式参数**
- 对未设置的重要参数也要列出（值填 `None`，说明填"未设置"）
- model 列标注 API 类型（Chat Completions / Responses）

### 3. 架构分层

ASCII 文本图，展示用户代码层 → agno.agent 层 → 模型层的数据流：

```markdown
## 架构分层

​```
用户代码层                agno.agent 层
┌──────────────────┐    ┌──────────────────────────────────┐
│ <文件名>.py      │    │ Agent._run()                     │
│                  │    │  ├ _messages.py                  │
│ <关键参数>       │───>│  │  get_system_message()          │
│                  │    │  │    → <处理逻辑概要>            │
│                  │    │  │                                │
│                  │    │  │  get_run_messages()            │
│                  │    │  │    <消息组装概要>              │
└──────────────────┘    └──────────────────────────────────┘
                                │
                                ▼
                        ┌──────────────┐
                        │ <Model类>    │
                        │ <model_id>   │
                        └──────────────┘
​```
```

规则：
- 左侧为用户代码层，展示关键参数
- 右侧为 agno 内部处理层，展示调用链
- 底部为模型层
- 如涉及工具、知识库等额外组件，在右侧或底部扩展

### 4. 核心组件解析

逐个分析该文件使用的 Agno 核心机制，附源码引用：

```markdown
## 核心组件解析

### <组件名1>

`<组件名>` 在 `<函数名>()`（`<文件>:<行号>`）中处理：

​```python
# 关键源码片段（带中文注释）
​```

<解释这段代码的作用和原理>

### <组件名2>
...
```

规则：
- 每个组件单独一个 `###` 子章节
- 引用 agno 源码时标注文件路径和行号
- 代码片段保留关键逻辑，添加中文注释
- 如有对比场景（如有/无某配置），用表格对比

### 5. System Prompt 组装

表格列出 system prompt 的所有组成部分及其在本文件中的状态。**序号必须与 `_messages.py` 中 `get_system_message()` 的步骤注释对齐**：

```markdown
## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 1 | `system_message`（自定义） | ... | 是/否 |
| 3.1 | `instructions` | ... | 是/否 |
| 3.1.1 | 模型指令（`get_instructions_for_model`） | ... | 是/否 |
| 3.2.1 | `markdown` | ... | 是/否 |
| 3.2.2 | `add_datetime_to_context` | ... | 是/否 |
| 3.2.3 | `add_location_to_context` | ... | 是/否 |
| 3.2.4 | `add_name_to_context` | ... | 是/否 |
| 3.3.1 | `description` | ... | 是/否 |
| 3.3.2 | `role` | ... | 是/否 |
| 3.3.3 | instructions 拼接 | ... | 是/否 |
| 3.3.4 | additional_information | ... | 是/否 |
| 3.3.5 | `_tool_instructions` | ... | 是/否 |
| 3.3.7 | `expected_output` | ... | 是/否 |
| 3.3.8 | `additional_context` | ... | 是/否 |
| 3.3.9 | `add_memories_to_context` | ... | 是/否 |

### 最终 System Prompt

​```text
<实际生成的 system prompt 内容>
​```
```

> 注意：步骤 2（`build_context=False`）仅在返回 None 时生效，一般不需列出。

### 6. 完整 API 请求

模拟实际发送给 LLM 的完整请求。根据模型类型选择正确的 API 格式：

**OpenAIResponses（Responses API）：**

```markdown
## 完整 API 请求

​```python
client.responses.create(
    model="<model_id>",
    input=[
        # 1. System Message（role_map: system → developer）
        {"role": "developer", "content": "..."},
        # 2. 其他消息（如有 additional_input、history 等）
        ...
        # N. 当前用户输入
        {"role": "user", "content": "..."}
    ],
    tools=[...],  # 如有
    stream=True,
    stream_options={"include_usage": True}
)
​```
```

**OpenAIChat（Chat Completions API）：**

```markdown
​```python
client.chat.completions.create(
    model="<model_id>",
    messages=[
        {"role": "system", "content": "..."},
        {"role": "user", "content": "..."}
    ],
    tools=[...],  # 如有
    stream=True,
    stream_options={"include_usage": True}
)
​```
```

规则：
- **Responses API** 用 `input` 参数，role 映射 `system→developer`
- **Chat Completions API** 用 `messages` 参数，role 保持 `system`
- 展示完整的消息数组
- 如有 tools 参数，展示完整的 tool schema
- 如涉及工具调用，展示多轮请求（第一轮 + 工具调用后的第二轮）
- 用注释标注每部分消息的来源
- 末尾用引用块补充说明

### 7. Mermaid 流程图

```markdown
## Mermaid 流程图

​```mermaid
flowchart TD
    A["用户代码<br/>agent.print_response(...)"] --> B["Agent._run()"]
    B --> C["get_system_message()"]

    subgraph System Prompt 组装
        C --> C1[...]
        ...
    end

    ... --> E["<Model>.invoke()"]
    E --> F["模型响应"]
    F --> G["流式输出"]

    style A fill:#e1f5fe
    style G fill:#e8f5e9
    style <关键节点> fill:#fff3e0
​```
```

规则：
- 用 `subgraph` 分组相关步骤
- 入口节点用 `#e1f5fe`（浅蓝），出口节点用 `#e8f5e9`（浅绿）
- 关键机制节点用 `#fff3e0`（浅橙）
- 如涉及条件分支，用菱形节点
- 如涉及工具调用循环，展示 agentic loop

### 8. 关键源码文件索引

```markdown
## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `<属性名>` L<行号> | <作用> |
| `agno/agent/_messages.py` | `<函数名>()` L<行号> | <作用> |
| ... | ... | ... |
```

规则：
- 只列与本文件核心机制直接相关的源码
- 标注行号（格式 `L<起始>-<结束>` 或 `L<行号>`）
- 按调用链顺序排列

## 分析工作流程

### 效率原则

- **不要使用 Agent 子进程**：Agent 子进程频繁出现 `API Error: Failed to parse JSON` 等错误，不可靠。**始终直接处理**。
- **并行读取**：第一步就并行读取所有目标 `.py` 文件 + 所有需要的源码文件，不要逐个串行读取
- **速查表优先**：先查「源码关键位置速查表」定位行号，仅在速查表不够时才 Grep
- **分批写入**：每批并行 Write 3 个 `.md` 文件（平衡效率和稳定性）
- **`_messages.py` 很大（~93KB，1300+ 行）**：必须分段读取，用 offset/limit 参数
- **源码只读一次**：agno 核心源码（agent.py、_messages.py、_response.py、responses.py）在同一会话中只需读取一次，后续目录直接复用

### 单文件模式

1. **并行 Read**（单次调用）：
   - 目标 `.py` 文件
   - `agno/agent/agent.py`（L67-350，Agent 类定义和所有属性）
   - `agno/agent/_messages.py`（L106-260，`get_system_message()`）
   - `agno/agent/_messages.py`（L1146-1345，`get_run_messages()`）
   - 如涉及工具：`agno/agent/_tools.py`（L105-195，`get_tools()`）
   - 如涉及特定模型：对应模型文件
   - 如涉及 output_model/parser_model：`agno/agent/_response.py`
2. **定位核心 Agno 特性**：识别 Agent 构造参数中的关键机制
3. **补充追踪**：仅对速查表未覆盖的特性进行 Grep
4. **组装文档**：按模板结构依次生成各章节
5. **Write** 输出到同目录的 `.md` 文件

### 批量模式（整个目录）

> **重要**：不使用 Agent 子进程，全部直接处理。

1. **Glob** 目标目录下所有 `.py` 文件 + 已有 `.md` 文件
2. **过滤**：排除 `__init__.py`、`__pycache__` 等非示例文件；跳过已有同名 `.md` 的文件
3. **并行 Read** 所有目标 `.py` 文件（单次调用）
4. **Read 源码**（如同一会话中未读取过）：并行读取 agent.py、_messages.py（分段）、responses.py 等
5. **补充 Grep**：对速查表未覆盖的新特性进行针对性 Grep
6. **分批 Write**：每批 3 个文件并行写入，直至全部完成
7. **确认**：Glob 检查所有 `.md` 文件已生成，输出总结表格

## Agno 源码关键位置速查表

> **重要**：以下行号基于当前代码库快照，源码更新后可能偏移。如果读取到的代码不匹配，用 Grep 重新定位。

### 核心调用链

```
Agent.print_response()          agent.py:1053
  └─ _cli.agent_print_response()  _cli.py:32
      ├─ stream=True  → print_response_stream()
      └─ stream=False → print_response()
          └─ _run._run()           _run.py:316
              ├─ 1. read_or_create_session()
              ├─ 5. get_tools()           _tools.py:105
              │     └─ determine_tools_for_model()  _tools.py:434
              ├─ 6. get_run_messages()    _messages.py:1146
              │     ├─ get_system_message()  _messages.py:106
              │     └─ get_user_message()
              └─ 9. Model.invoke() / invoke_stream()
```

### Agent 属性定义（agent.py）

| 属性 | 行号 | 说明 |
|------|------|------|
| `Agent` 类定义 | L67 | `@dataclass(init=False)` |
| `model` | L70 | 模型实例 |
| `name` | L72 | Agent 名称 |
| `tools` | L159 | 工具列表 |
| `tool_call_limit` | L162 | 工具调用次数限制 |
| `tool_choice` | L169 | 工具选择策略 |
| `system_message` | L217 | 自定义 system message |
| `build_context` | L223 | 是否构建上下文 |
| `description` | L227 | Agent 描述 |
| `instructions` | L229 | 指令（str/list/callable） |
| `use_instruction_tags` | L231 | 是否用 XML 标签包裹指令 |
| `expected_output` | L233 | 期望输出 |
| `additional_context` | L235 | 额外上下文 |
| `markdown` | L237 | markdown 格式化 |
| `add_name_to_context` | L239 | 添加名称到上下文 |
| `add_datetime_to_context` | L242 | 添加时间到上下文 |
| `add_location_to_context` | L245 | 添加位置到上下文 |
| `timezone_identifier` | L247 | 自定义时区 |
| `additional_input` | L261 | 额外消息（few-shot 等） |
| `input_schema` | L278 | 输入模式验证 |
| `output_schema` | L281 | 结构化输出 |
| `parser_model` | L283 | 解析模型 |
| `parser_model_prompt` | L285 | 解析模型 prompt |
| `output_model` | L287 | 输出模型 |
| `output_model_prompt` | L289 | 输出模型 prompt |
| `save_response_to_file` | L298 | 保存响应到文件 |
| `role` | L312 | Agent 角色（团队中） |
| `introduction` | L221 | 初始问候消息 |
| `system_message_role` | L219 | system 消息角色 |
| `add_history_to_context` | L127 | 历史消息开关 |
| `max_tool_calls_from_history` | L133 | 历史工具调用限制 |
| `db` | L123 | 数据库配置 |
| `print_response()` | L1053 | 用户调用入口 |

### get_system_message() 步骤索引（_messages.py）

| 步骤号 | 行号 | 内容 |
|--------|------|------|
| 函数签名 | L106 | `get_system_message()` |
| 1. 自定义 system_message | L130 | 如设置则直接使用 |
| 2. build_context=False | L155 | 返回 None |
| 3.1 instructions 解析 | L163-174 | str/list/callable → List[str] |
| 3.1.1 模型指令 | L177-179 | `get_instructions_for_model()` |
| 3.2.1 markdown | L184-185 | 格式化指令 |
| 3.2.2 add_datetime | L187-202 | 当前时间 |
| 3.2.3 add_location | L205-221 | 当前位置 |
| 3.2.4 add_name | L224-225 | Agent 名称 |
| 3.3.1 description | L230-231 | 追加描述 |
| 3.3.2 role | L233-234 | 追加角色 |
| 3.3.3 instructions 拼接 | L236-250 | 写入 system message |
| 3.3.4 additional_information | L252-256 | 附加信息 |
| 3.3.5 _tool_instructions | L258-260 | 工具使用说明 |
| 3.3.7 expected_output | L271-272 | 期望输出 |
| 3.3.8 additional_context | L274-275 | 额外上下文 |
| 3.3.9 memories | L282-320 | 用户记忆 |

### get_run_messages() 步骤索引（_messages.py）

| 步骤号 | 行号 | 内容 |
|--------|------|------|
| 函数签名 | L1146 | `get_run_messages()` |
| 1. system message | L1192-1201 | 调用 get_system_message() |
| 2. additional_input | L1204-1228 | 额外消息 |
| 3. history | L1231-1262 | 历史消息 |
| 4. user message | L1265-1343 | 用户消息（多种输入类型） |

### 模型文件速查

| 模型类 | 文件 | 类定义 | invoke() | role_map |
|--------|------|--------|----------|----------|
| `OpenAIResponses` | `models/openai/responses.py` | L31 | L574 | L84（system→developer） |
| `OpenAIChat` | `models/openai/chat.py` | 顶部 | 搜索 `def invoke` | system→system |

### 响应处理速查（_response.py）

| 函数 | 行号 | 说明 |
|------|------|------|
| `parse_response_with_parser_model()` | L364 | parser_model 解析入口 |
| `parse_response_with_parser_model_stream()` | L460 | parser_model 流式版 |
| `generate_response_with_output_model()` | L597 | output_model 调用入口 |
| `generate_response_with_output_model_stream()` | L623 | output_model 流式版 |
| `model_should_return_structured_output()` | L860 | 判断是否使用 structured outputs |
| `get_response_format()` | L872 | 获取 response_format |

### 消息构造速查（_messages.py 补充）

| 函数 | 行号 | 说明 |
|------|------|------|
| `get_messages_for_parser_model()` | L1591 | 构造 parser_model 的消息 |
| `get_messages_for_parser_model_stream()` | L1616 | 流式版 |
| `get_messages_for_output_model()` | L1641 | 构造 output_model 的消息 |

### 工具文件速查

| 工具类 | 文件 | 行号 |
|--------|------|------|
| `_tools.get_tools()` | `agent/_tools.py` | L105 |
| `_tools.determine_tools_for_model()` | `agent/_tools.py` | L434 |
| `DuckDuckGoTools` | `tools/duckduckgo.py` | L6 |
| `WebSearchTools` | `tools/websearch.py` | L16 |
| `WebSearchTools.web_search()` | `tools/websearch.py` | L74 |
| `HackerNewsTools` | `tools/hackernews.py` | 顶部 |
| `YFinanceTools` | `tools/yfinance.py` | 顶部 |

### 其他速查

| 函数/类 | 文件 | 行号 | 说明 |
|---------|------|------|------|
| `execute_instructions()` | `utils/agent.py` | L949 | 执行 callable instructions |
| `filter_tool_calls()` | `utils/message.py` | L10 | 过滤历史工具调用 |
| `save_run_response_to_file()` | `agent/_run.py` | L4295 | 保存响应到文件 |
| `cleanup_and_store()` | `agent/_run.py` | L4348 | 运行后清理存储 |

### 追踪技巧

- **不要反复 Grep 已知位置**：先查速查表，直接 Read 对应行号范围
- **`_messages.py` 很大（~93KB）**：必须用 offset/limit 分段读取，不要一次读取全文
- 关注 `get_system_message()` 的分段注释（`# 3.1`, `# 3.2` 等）
- 记录行号时使用 `L<行号>` 格式
- 如果速查表行号与实际不符，用 `Grep "def get_system_message"` 等快速重新定位
- **同一会话内**，agno 源码只需读取一次，后续目录生成时直接复用已有知识

## 错误处理

### Write/Read 工具报错

- 如果 Write 输出文档时报错，直接重试（最多 2 次）
- 如果 Read 源码时报错，直接重试（最多 2 次）
- 所有重试均为即时重试，无需等待间隔

### 关于 Agent 子进程

> **禁止使用 Agent 子进程。** 实战中 Agent 子进程多次出现 `API Error: Failed to parse JSON` 等不可恢复错误，即使重试 3 次也无法解决。直接处理（主进程 Read + Write）**零失败**，效率更高。

## 注意事项

- **不要虚构行号**：必须通过实际 Read/Grep 确认源码行号
- **不要遗漏配置项**：核心配置一览表要列出所有显式参数
- **API 请求要准确**：根据实际的模型类型确定 role（`developer` for OpenAI Chat，`user` for Responses API 等）
- **Mermaid 节点 ID 不能有空格**：用驼峰或下划线命名
- **跳过已存在的 .md 文件**：批量模式下，如果同名 .md 已存在则跳过
