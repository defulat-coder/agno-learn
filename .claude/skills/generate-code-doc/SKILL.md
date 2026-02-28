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
| fmt | `resolve_in_context` 变量替换 | ... | 是/否 |
| 3.3.7 | `expected_output` | ... | 是/否 |
| 3.3.8 | `additional_context` | ... | 是/否 |
| 3.3.9 | `add_memories_to_context` | ... | 是/否 |
| 3.3.10 | `add_culture_to_context` | ... | 是/否 |
| 3.3.11 | `add_session_summary_to_context` | ... | 是/否 |
| 3.3.12 | `add_learnings_to_context` | ... | 是/否 |
| 3.3.13 | `search_knowledge` instructions | ... | 是/否 |
| 3.3.14 | 模型 system message | ... | 是/否 |
| 3.3.15 | JSON output prompt | ... | 是/否 |
| 3.3.16 | response model format prompt | ... | 是/否 |
| 3.3.17 | `add_session_state_to_context` | ... | 是/否 |

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
- **源码只读一次**：agno 核心源码（agent.py、_messages.py、_response.py、responses.py、run/base.py、run/agent.py、_managers.py）在同一会话中只需读取一次，后续目录直接复用
- **`memory/manager.py` 很大（~70KB）**：必须用 offset/limit 分段读取，建议分 3 段：L1-200（类定义 + 基础方法）、L400-560（update_memory_task/agentic 流程）、L958-990（get_system_message）
- **`_messages.py` 需要读取 3-5 个区间**：L56-98（format_message_with_state_variables）、L106-440（get_system_message 含所有步骤）、L1146-1345（get_run_messages）。session/state 类文件经常用到 L260-440 的后半段步骤，不要遗漏。knowledge 类文件还需 L905-954（Traditional RAG 注入）和 L1665-1761（get_relevant_docs_from_knowledge）
- **`knowledge/knowledge.py` 很大（~3400 行）**：必须用 offset/limit 分段读取，关键区间：L41-55（类定义）、L507-546（search）、L2879-2933（build_context + 指令模板）、L3303-3324（retrieve）

### 单文件模式

1. **并行 Read**（单次调用）：
   - 目标 `.py` 文件
   - `agno/agent/agent.py`（L67-350，Agent 类定义和所有属性）
   - `agno/agent/_messages.py`（L106-260，`get_system_message()`）
   - `agno/agent/_messages.py`（L1146-1345，`get_run_messages()`）
   - 如涉及工具：`agno/agent/_tools.py`（L105-195，`get_tools()`）
   - 如涉及 Knowledge/知识库：`agno/knowledge/knowledge.py`（L41-55 类定义 + L507-546 search + L2879-2933 build_context + L3303-3324 retrieve）
   - 如涉及 search_knowledge 或 knowledge_retriever：`agno/agent/_default_tools.py`（L103-282，create_knowledge_search_tool）
   - 如涉及 add_knowledge_to_context（Traditional RAG）：`agno/agent/_messages.py`（L905-954，get_user_message 中的预检索注入）
   - 如涉及 knowledge_retriever：`agno/agent/_messages.py`（L1665-1761，get_relevant_docs_from_knowledge）
   - 如涉及 knowledge_filters/enable_agentic_knowledge_filters：`agno/filters.py`（L46-106）+ `agno/utils/knowledge.py`（全文）
   - 如涉及 ReasoningTools：`agno/tools/reasoning.py`（全文，~291 行）
   - 如涉及 callable factory（tools/knowledge/members 为函数）：`agno/utils/callables.py`（全文，~613 行）
   - 如涉及 Toolkit 类：`agno/tools/toolkit.py`（L10-80）
   - 如涉及 Literal 类型参数：`agno/utils/json_schema.py`（L124-143）
   - 如涉及 Team：`agno/team/team.py`
   - 如涉及特定模型：对应模型文件
   - 如涉及 output_model/parser_model：`agno/agent/_response.py`
   - 如涉及 session_state 模板变量/resolve_in_context：`agno/agent/_messages.py`（L56-98，`format_message_with_state_variables`）
   - 如涉及 enable_agentic_state：`agno/agent/_default_tools.py`（L347-380）
   - 如涉及 tool_hooks：`agno/tools/function.py`（L898-974，`_build_hook_args` + `_build_nested_execution_chain`）
   - 如涉及 search_session_history：`agno/agent/_default_tools.py`（L411-480）
   - 如涉及 enable_session_summaries：`agno/session/summary.py`（L22-106）
   - 如涉及 stream_events/RunCompletedEvent：`agno/run/agent.py`（L134-278）
   - 如涉及 RunContext（工具函数参数）：`agno/run/base.py`（L16-33）
   - 如涉及 LearningMachine/learning：`agno/learn/machine.py`（L53-120 类定义 + L350-465 build_context/get_tools + L498-540 process）、`agno/learn/config.py`（全文，~464 行）
   - 如涉及 enable_agentic_memory/memory_manager/MemoryManager：`agno/memory/manager.py`（L44-100 类定义 + L481-517 update_memory_task + L958-986 get_system_message）、`agno/agent/_default_tools.py`（L38-75）
   - 如涉及后台任务编排（memory/learning/culture 自动提取）：`agno/agent/_managers.py`（L29-50 make_memories + L177-210 start_memory_future + L487-521 start_learning_future）
   - 如涉及 pre_hooks/post_hooks/Guardrail：`agno/agent/_hooks.py`（全文）+ `agno/utils/hooks.py`（全文）+ `agno/exceptions.py`（L122-173）
   - 如涉及内置 Guardrail（PIIDetectionGuardrail/OpenAIModerationGuardrail/PromptInjectionGuardrail）：对应 `agno/guardrails/*.py`（均为小文件，可全文读取）
   - 如涉及自定义 BaseGuardrail 子类：`agno/guardrails/base.py`（全文，~20 行）
   - 如涉及 RunInput/RunOutput（hook 参数类型）：`agno/run/agent.py`（L29-65 RunInput + L581-624 RunOutput）
2. **定位核心 Agno 特性**：识别 Agent 构造参数中的关键机制
3. **补充追踪**：仅对速查表未覆盖的特性进行 Grep
4. **组装文档**：按模板结构依次生成各章节
5. **Write** 输出到同目录的 `.md` 文件

### 批量模式（整个目录）

> **重要**：不使用 Agent 子进程，全部直接处理。

1. **Glob** 目标目录下所有 `.py` 文件 + 已有 `.md` 文件
2. **过滤**：排除 `__init__.py`、`__pycache__` 等非示例文件；跳过已有同名 `.md` 的文件
3. **并行 Read** 所有目标 `.py` 文件（单次调用）
4. **扫描特性**：浏览所有 `.py` 文件内容，识别使用的 Agno 特性（callable factory、Toolkit、Literal、Team 等），据此决定需要额外读取的源码文件
5. **Read 源码**（如同一会话中未读取过）：
   - 基础（必读）：agent.py（L67-350）、_messages.py（L106-260 + L1146-1345）
   - 按需追加（根据步骤 4 的扫描结果）：
     - 有工具 → `_tools.py`（L105-195 + L350-477）
     - 有 Knowledge/知识库 → `knowledge/knowledge.py`（L41-55 + L507-546 + L2879-2933 + L3303-3324）
     - 有 search_knowledge / knowledge_retriever → `_default_tools.py`（L103-282）+ `_messages.py` L1665-1761
     - 有 add_knowledge_to_context → `_messages.py` L905-954
     - 有 knowledge_filters / enable_agentic_knowledge_filters → `filters.py` L46-106 + `utils/knowledge.py`（全文）
     - 有 ReasoningTools → `tools/reasoning.py`（全文）
     - 有 callable factory → `utils/callables.py`（全文）
     - 有 Toolkit 子类 → `tools/toolkit.py`（L10-80）
     - 有 Literal 参数 → `utils/json_schema.py`（L124-143）
     - 有 Team → `team/team.py`
     - 有 tool_call_limit/tool_choice → `_run.py`（L490-510）
     - 有 session_state 模板变量 → `_messages.py`（L56-98）
     - 有 enable_agentic_state → `_tools.py`（L165-171）+ `_default_tools.py`（L347-380）
     - 有 tool_hooks → `tools/function.py`（L898-974）
     - 有 search_session_history → `_tools.py`（L143-148）+ `_default_tools.py`（L411-480）
     - 有 enable_session_summaries → `session/summary.py`（L22-106）
     - 有 stream_events / RunCompletedEvent → `run/agent.py`（L134-278）
     - 有 RunContext 用法 → `run/base.py`（L16-33）
     - 有 `learning` / `LearningMachine` → `learn/machine.py`（L53-120 + L350-465 + L498-540）+ `learn/config.py`（全文）
     - 有 `enable_agentic_memory` / `memory_manager` / `MemoryManager` → `memory/manager.py`（L44-100 + L481-517 + L958-986）+ `_default_tools.py` L38-75
     - 有 `update_memory_on_run` / `add_memories_to_context` → `_managers.py` L29-50 + L177-210 + `_messages.py` L282-320
     - 有 `pre_hooks` / `post_hooks` / `BaseGuardrail` → `_hooks.py`（全文）+ `utils/hooks.py`（全文）+ `exceptions.py` L122-173
     - 有 `PIIDetectionGuardrail` → `guardrails/pii.py`（全文）
     - 有 `OpenAIModerationGuardrail` → `guardrails/openai.py`（全文）
     - 有 `PromptInjectionGuardrail` → `guardrails/prompt_injection.py`（全文）
     - 有自定义 `BaseGuardrail` 子类 → `guardrails/base.py`（全文）
     - 有 `RunInput` / `RunOutput`（hook 参数） → `run/agent.py` L29-65 + L581-624
6. **补充 Grep**：对速查表未覆盖的新特性进行针对性 Grep
7. **TaskCreate**：为每个待生成的文件创建 task，便于跟踪进度和断点续做
8. **分批 Write**：每批 3 个文件并行写入，完成后更新 task 状态，直至全部完成
9. **确认**：Glob 检查所有 `.md` 文件已生成，输出总结表格

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
              ├─ 3. normalize_pre/post_hooks()  _run.py:1250-1256
              │     └─ normalize_pre_hooks()     utils/hooks.py:70
              │     └─ normalize_post_hooks()    utils/hooks.py:113
              ├─ 4. execute_pre_hooks()   _hooks.py:43
              │     └─ filter_hook_args()  utils/hooks.py:156
              │     └─ InputCheckError? → 中断运行
              ├─ 5. get_tools()           _tools.py:105
              │     ├─ resolve_callable_tools()  callables.py:213
              │     │     ├─ is_callable_factory()
              │     │     ├─ _compute_cache_key()
              │     │     └─ invoke_callable_factory()
              │     └─ determine_tools_for_model()  _tools.py:434
              │           └─ parse_tools()  _tools.py:350
              ├─ 6. get_run_messages()    _messages.py:1146
              │     ├─ get_system_message()  _messages.py:106
              │     └─ get_user_message()
              ├─ 9. Model.response(       _run.py:501
              │       tool_choice=...,    _run.py:504
              │       tool_call_limit=... _run.py:505
              │     )
              └─ 10. execute_post_hooks()  _hooks.py:266
                    └─ OutputCheckError? → 设置 error 状态

Team.print_response()
  └─ Team._run()
        ├─ resolve_callable_members()  callables.py:374
        ├─ 委派任务给成员 Agent
        └─ 各成员 Agent._run()（独立运行）
```

### Agent 属性定义（agent.py）

| 属性 | 行号 | 说明 |
|------|------|------|
| `Agent` 类定义 | L67 | `@dataclass(init=False)` |
| `model` | L70 | 模型实例 |
| `name` | L72 | Agent 名称 |
| `session_state` | L84 | 默认 session state 字典 |
| `add_session_state_to_context` | L86 | 将 session_state 注入 system prompt |
| `enable_agentic_state` | L88 | 启用内置 update_session_state 工具 |
| `overwrite_db_session_state` | L90 | 是否覆盖（默认合并）DB 中的状态 |
| `search_session_history` | L94 | 启用跨会话搜索工具 |
| `num_history_sessions` | L95 | 搜索会话数限制 |
| `enable_session_summaries` | L97 | 启用自动摘要 |
| `add_session_summary_to_context` | L99 | 摘要注入 system prompt |
| `session_summary_manager` | L101 | 自定义摘要管理器 |
| `search_knowledge` | L195 | 启用 Agentic RAG 搜索工具（默认 `True`） |
| `add_search_knowledge_instructions` | L197 | 是否向 system prompt 注入搜索指令（默认 `True`） |
| `update_knowledge` | L199 | 启用知识库更新工具 |
| `tools` | L159 | 工具列表（List 或 Callable 工厂） |
| `tool_call_limit` | L162 | 工具调用次数限制 |
| `tool_choice` | L169 | 工具选择策略（none/auto/指定函数） |
| `tool_hooks` | L172 | 工具调用中间件 |
| `pre_hooks` | L176 | 前置 hook 列表（Callable / BaseGuardrail / BaseEval） |
| `post_hooks` | L178 | 后置 hook 列表（Callable / BaseGuardrail / BaseEval） |
| `_run_hooks_in_background` | L180 | 后台执行 hooks（AgentOS 设置） |
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
| `cache_callables` | L352 | 控制是否缓存 callable factory 结果（默认 True） |
| `role` | L312 | Agent 角色（团队中） |
| `introduction` | L221 | 初始问候消息 |
| `system_message_role` | L219 | system 消息角色 |
| `add_history_to_context` | L127 | 历史消息开关 |
| `num_history_runs` | L129 | 限制历史运行次数 |
| `num_history_messages` | L131 | 限制历史消息数量 |
| `max_tool_calls_from_history` | L133 | 历史工具调用限制 |
| `knowledge` | L136 | Knowledge 实例（或 Callable 工厂） |
| `knowledge_filters` | L139 | 静态过滤器（Dict 或 List[FilterExpr]） |
| `enable_agentic_knowledge_filters` | L141 | 让模型动态选择过滤条件 |
| `add_knowledge_to_context` | L142 | Traditional RAG：预注入上下文到用户消息 |
| `knowledge_retriever` | L148 | 自定义检索函数（替代 Knowledge.search） |
| `references_format` | L149 | 引用格式（`Literal["json", "yaml"]`，默认 `"json"`） |
| `memory_manager` | L111 | MemoryManager 实例 |
| `enable_agentic_memory` | L113 | 启用代理式记忆（注册 update_user_memory 工具） |
| `update_memory_on_run` | L115 | 每次运行后自动提取记忆 |
| `add_memories_to_context` | L119 | 将用户记忆注入 system prompt |
| `db` | L123 | 数据库配置 |
| `resolve_in_context` | L249 | 启用 session_state/dependencies 模板变量替换（默认 True） |
| `learning` | L253 | LearningMachine 统一学习系统（bool 或 LearningMachine 实例） |
| `add_learnings_to_context` | L255 | 将学习上下文注入 system prompt（默认 True） |
| `store_history_messages` | L213 | 控制是否存储历史消息（默认 False=线性增长） |
| `stream` | L302 | 流式响应 |
| `stream_events` | L304 | 流式事件模式 |
| `get_session_state()` | L939 | 获取当前 session_state |
| `update_session_state()` | L945 | 手动更新 session_state |
| `get_chat_history()` | L1005 | 获取聊天历史 |
| `get_session_summary()` | L1013 | 获取会话摘要 |
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
| 3.3.10 cultural knowledge | L322-381 | 文化知识 |
| 3.3.11 session summary | L384-392 | 会话摘要（`add_session_summary_to_context`） |
| 3.3.12 learnings | L395-402 | 学习上下文 |
| 3.3.13 search_knowledge instructions | L404-413 | 知识库搜索指令 |
| 3.3.14 model system message | L416-418 | 模型级 system message |
| 3.3.15 JSON output prompt | L420-430 | JSON 格式化 prompt |
| 3.3.16 parser_model format | L433-434 | parser_model 格式 prompt |
| 3.3.17 session_state to context | L437-438 | `add_session_state_to_context` XML 注入 |
| fmt `format_message_with_state_variables` | L262-268 | 在 3.3.5 后、3.3.7 前执行变量替换 |

### format_message_with_state_variables()（_messages.py L56）

将 instructions/system_message 中的 `{var}` 模板变量替换为 `session_state`、`dependencies`、`metadata`、`user_id` 中的值。使用 `string.Template.safe_substitute()`，未匹配的变量保持原样。受 `resolve_in_context`（默认 True）控制。

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
| Traditional RAG 预检索注入 | L905-954 | `add_knowledge_to_context=True` 时预检索并注入 `<references>` 到用户消息 |
| `get_relevant_docs_from_knowledge()` | L1665 | 同步检索文档（knowledge_retriever 优先 → Knowledge.retrieve 回退） |
| `aget_relevant_docs_from_knowledge()` | L1764 | 异步版 |
| `get_messages_for_parser_model()` | L1591 | 构造 parser_model 的消息 |
| `get_messages_for_parser_model_stream()` | L1616 | 流式版 |
| `get_messages_for_output_model()` | L1641 | 构造 output_model 的消息 |

### 工具文件速查

| 工具类 | 文件 | 行号 |
|--------|------|------|
| `_tools.get_tools()` | `agent/_tools.py` | L105 |
| 注册 knowledge 搜索工具 | `agent/_tools.py` | L176-186 |
| 注册 enable_agentic_memory 工具 | `agent/_tools.py` | L150-151 |
| 注册 LearningMachine 工具 | `agent/_tools.py` | L154-160 |
| `_tools.parse_tools()` | `agent/_tools.py` | L350-431 |
| `_tools.determine_tools_for_model()` | `agent/_tools.py` | L434 |
| `Toolkit` 基类 | `tools/toolkit.py` | L10 |
| `Toolkit.__init__()` | `tools/toolkit.py` | L15-80 |
| `Function` | `tools/function.py` | L40 |
| `Function.from_callable()` | `tools/function.py` | 搜索 `def from_callable` |
| `DuckDuckGoTools` | `tools/duckduckgo.py` | L6 |
| `WebSearchTools` | `tools/websearch.py` | L16 |
| `WebSearchTools.web_search()` | `tools/websearch.py` | L74 |
| `HackerNewsTools` | `tools/hackernews.py` | 顶部 |
| `YFinanceTools` | `tools/yfinance.py` | 顶部 |
| `ReasoningTools` | `tools/reasoning.py` | L10 |
| `ReasoningTools.think()` | `tools/reasoning.py` | L51 |
| `ReasoningTools.analyze()` | `tools/reasoning.py` | L117 |
| `ReasoningTools.DEFAULT_INSTRUCTIONS` | `tools/reasoning.py` | L193 |

### Callable Factory 速查（utils/callables.py）

> 工具/知识库/团队成员均可用 callable 工厂动态解析，共享此文件的逻辑。

| 函数 | 行号 | 说明 |
|------|------|------|
| `is_callable_factory()` | L40 | 判断值是否为工厂函数（排除 Toolkit/Function/type） |
| `invoke_callable_factory()` | L60 | 签名检查 + 参数注入（agent/run_context/session_state） |
| `ainvoke_callable_factory()` | L104 | 异步版本 |
| `_compute_cache_key()` | L133 | 缓存 key 优先级：custom_key_fn > user_id > session_id > None |
| `resolve_callable_tools()` | L213 | 解析 callable tools 工厂 |
| `resolve_callable_knowledge()` | L294 | 解析 callable knowledge 工厂 |
| `resolve_callable_members()` | L374 | 解析 callable members 工厂（Team 用） |
| `get_resolved_tools()` | L595 | 获取解析后的工具（run_context > entity） |
| `get_resolved_members()` | L605 | 获取解析后的成员 |
| `clear_callable_cache()` | L453 | 清除缓存 |

**签名注入参数映射（invoke_callable_factory L79-89）：**

| 工厂函数参数名 | 注入内容 | 说明 |
|---------------|---------|------|
| `agent` | Agent 实例 | |
| `team` | Team 实例 | |
| `run_context` | 完整 RunContext 对象 | 含 user_id/session_id/session_state |
| `session_state` | `run_context.session_state or {}` | 仅 state 字典 |

**缓存行为（受 `cache_callables` 控制）：**

| `cache_callables` | 缓存行为 | 适用场景 |
|-------------------|---------|---------|
| `True`（默认） | 按 cache_key 缓存，同 key 不重复调用 | 角色固定，同用户工具不变 |
| `False` | 每次运行重新调用工厂 | session_state 频繁变化 |

### JSON Schema 速查（utils/json_schema.py）

| 函数 | 行号 | 说明 |
|------|------|------|
| `get_json_schema_for_arg()` | L124 | 类型注解 → JSON Schema 转换入口 |
| Literal 处理 | L127-143 | `Literal["a","b"]` → `{"type":"string","enum":["a","b"]}` |

**Literal 类型转换规则（L127-143）：**

| Literal 值类型 | JSON Schema |
|---------------|-------------|
| 全 `str` | `{"type": "string", "enum": [...]}` |
| 全 `bool` | `{"type": "boolean", "enum": [...]}` |
| 全 `int` | `{"type": "integer", "enum": [...]}` |
| `int`+`float` | `{"type": "number", "enum": [...]}` |
| 混合 | `{"enum": [...]}` |

### Team 速查

| 类/函数 | 文件 | 说明 |
|---------|------|------|
| `Team` | `team/team.py` | 团队类，`members` 支持 List 或 Callable |
| `members` | `team/team.py` | 成员列表或工厂函数 |
| `cache_callables` | `team/team.py` | 控制是否缓存工厂结果 |

### RunContext 速查（run/base.py）

| 类/字段 | 行号 | 说明 |
|---------|------|------|
| `RunContext` | L16 | 运行上下文 dataclass |
| `run_id` | L17 | 运行 ID |
| `session_id` | L18 | 会话 ID |
| `user_id` | L19 | 用户 ID |
| `session_state` | L27 | 会话状态字典（引用传递，工具函数可直接修改） |

### 事件类型速查（run/agent.py）

| 类 | 行号 | 说明 |
|----|------|------|
| `RunEvent` | L134 | 事件类型枚举 |
| `RunCompletedEvent` | L261 | 运行完成事件，含 session_state, metrics, content |

### Default Tools 速查（agent/_default_tools.py）

| 函数 | 行号 | 说明 |
|------|------|------|
| `get_update_user_memory_function()` | L38 | 代理式记忆工具工厂（enable_agentic_memory 用） |
| `update_user_memory()` | L39 | 调用 MemoryManager.update_memory_task() |
| `create_knowledge_search_tool()` | L103 | 创建 search_knowledge_base 工具（统一入口） |
| `_format_results()` | L118 | 按 references_format（json/yaml）序列化文档 |
| `_track_references()` | L130 | 记录检索引用到 run_response |
| `_resolve_filters()` | L141 | 合并代理式过滤器和用户过滤器 |
| `search_knowledge_base_with_filters()` | L156 | 带过滤器的搜索工具（enable_agentic_knowledge_filters 用） |
| `search_knowledge_base()` | L224 | 无过滤器的搜索工具（默认） |
| `update_session_state_tool()` | L347 | 内置状态更新工具实现（逐 key 合并） |
| `make_update_session_state_entrypoint()` | L366 | 绑定 agent 的闭包工厂 |
| `get_previous_sessions_messages_function()` | L411 | 跨会话搜索工具工厂 |
| `get_previous_session_messages()` | L425 | 搜索工具实际实现（按 user_id 过滤） |

### Knowledge 速查（knowledge/knowledge.py）

> Knowledge 类是知识库的核心抽象，封装向量数据库的插入、搜索和上下文构建。

| 函数/类 | 行号 | 说明 |
|---------|------|------|
| `Knowledge` | L41 | 知识库 dataclass（继承 RemoteKnowledge） |
| `vector_db` | L46 | 向量数据库实例 |
| `max_results` | L48 | 默认返回文档数（10） |
| `insert()` | L90 | 同步插入内容（path/url/text_content） |
| `ainsert_many()` | 搜索 | 异步批量插入 |
| `search()` | L507 | 向量搜索入口（代理到 vector_db.search） |
| `retrieve()` | L3303 | 检索接口（代理到 search，用于 add_knowledge_to_context） |
| `build_context()` | L2908 | 构建 system prompt 中的搜索指令（步骤 3.3.13 调用） |
| `_SEARCH_KNOWLEDGE_INSTRUCTIONS` | L2879 | 搜索指令常量文本 |
| `_AGENTIC_FILTER_INSTRUCTION_TEMPLATE` | L2885 | 代理式过滤器指令模板 |
| `_get_agentic_filter_instructions()` | L2903 | 生成过滤器使用指令（含示例） |
| `get_valid_filters()` | 搜索 | 获取向量数据库合法过滤键集合 |

**Knowledge.build_context() 输出结构：**

```
<knowledge_base>
搜索指令（_SEARCH_KNOWLEDGE_INSTRUCTIONS）
[如 enable_agentic_filters] 过滤器指令（_AGENTIC_FILTER_INSTRUCTION_TEMPLATE）
</knowledge_base>
```

### 知识库检索速查（_messages.py）

| 函数 | 行号 | 说明 |
|------|------|------|
| `_get_resolved_knowledge()` | L44 | 获取解析后的 Knowledge（优先 run_context） |
| `get_relevant_docs_from_knowledge()` | L1665 | 同步检索文档（knowledge_retriever 优先 → Knowledge.retrieve 回退） |
| `aget_relevant_docs_from_knowledge()` | L1764 | 异步版 |
| Traditional RAG 注入（get_user_message 内） | L905-954 | `add_knowledge_to_context=True` 时预检索并注入 `<references>` |

**knowledge_retriever 参数注入映射（L1716-1732）：**

| 检索函数参数名 | 注入内容 | 说明 |
|---------------|---------|------|
| `query` | 搜索查询字符串 | 必需（始终传入） |
| `num_documents` | 文档数量限制 | 始终传入 |
| `agent` | Agent 实例 | 按签名注入 |
| `filters` | 知识过滤器 | 按签名注入 |
| `run_context` | RunContext 对象 | 按签名注入 |
| `dependencies` | 依赖字典（向后兼容） | 当无 `run_context` 参数时 |

### 过滤器速查（filters.py + utils/knowledge.py）

| 类/函数 | 文件 | 行号 | 说明 |
|---------|------|------|------|
| `FilterExpr` | `filters.py` | L46 | 过滤表达式基类（支持 `|` `&` `~` 运算符） |
| `EQ` | `filters.py` | L86 | 等值过滤器 `EQ("key", "value")` |
| `IN` | `filters.py` | L109 | 包含过滤器 |
| `GT` / `LT` | `filters.py` | 搜索 | 大于/小于过滤器 |
| `AND` / `OR` / `NOT` | `filters.py` | 搜索 | 逻辑组合过滤器 |
| `get_agentic_or_user_search_filters()` | `utils/knowledge.py` | L7 | 过滤器优先级（用户 > 代理） |

### LearningMachine 速查（learn/machine.py）

| 函数/类 | 行号 | 说明 |
|---------|------|------|
| `LearningMachine` | L53 | 统一学习系统协调器 dataclass |
| `_initialize_stores()` | L111 | 懒加载初始化所有配置的存储 |
| `_resolve_store()` | L164 | 根据 bool/Config/Store 创建具体存储实例 |
| `_create_user_profile_store()` | L198 | 创建 UserProfileStore（继承 db/model） |
| `build_context()` | L350 | 构建学习上下文字符串（调用 recall → _format_results） |
| `get_tools()` | L420 | 获取 AGENTIC 模式工具（遍历各存储的 get_tools） |
| `process()` | L498 | 运行后学习提取（遍历各存储的 process） |
| `recall()` | L572 | 从各存储检索原始数据 |
| `_format_results()` | L658 | 格式化检索结果为上下文字符串 |

### LearningConfig 速查（learn/config.py）

| 类 | 说明 |
|----|------|
| `LearningMode` | 枚举：ALWAYS（自动）、AGENTIC（工具）、PROPOSE（审批）、HITL（人类在环） |
| `UserProfileConfig` | 用户画像配置（mode, db, model） |
| `UserMemoryConfig` | 用户记忆配置 |
| `SessionContextConfig` | 会话上下文配置 |
| `EntityMemoryConfig` | 实体记忆配置（namespace） |
| `LearnedKnowledgeConfig` | 学到的知识配置（knowledge） |
| `DecisionLogConfig` | 决策日志配置 |

### MemoryManager 速查（memory/manager.py）

> `memory/manager.py` 约 70KB，必须用 offset/limit 分段读取。

| 函数/类 | 行号 | 说明 |
|---------|------|------|
| `MemoryManager` | L44 | 记忆管理器 dataclass |
| `__init__()` | L76 | 构造函数（model 支持 str 自动转换） |
| `get_user_memories()` | L165 | 获取用户记忆列表 |
| `add_user_memory()` | L211 | 添加单条记忆 |
| `create_user_memories()` | L368 | 从对话消息批量创建记忆（update_memory_on_run 用） |
| `update_memory_task()` | L481 | 工具调用入口（enable_agentic_memory 用） |
| `run_memory_task()` | 内部 | 构建独立模型调用执行记忆 CRUD |
| `search_user_memories()` | L588 | 搜索记忆（last_n/first_n/agentic） |
| `get_system_message()` | L958 | 记忆管理器的 system prompt |
| `determine_tools_for_model()` | L938 | 将记忆工具转换为 Function 列表 |

### 后台任务编排速查（agent/_managers.py）

> `_managers.py` 统一管理 memory、learning、cultural knowledge 的后台提取。

| 函数 | 行号 | 说明 |
|------|------|------|
| `make_memories()` | L29 | 同步创建记忆（update_memory_on_run 模式） |
| `start_memory_future()` | L177 | 后台线程启动记忆提取（非 agentic 时） |
| `get_user_memories()` | L212 | 获取用户记忆（自动初始化 memory_manager） |
| `make_cultural_knowledge()` | L259 | 同步创建文化知识 |
| `process_learnings()` | L398 | 同步处理学习提取（调用 _learning.process） |
| `start_learning_future()` | L487 | 后台线程启动学习提取 |
| `astart_learning_task()` | L445 | 异步任务版学习提取 |

**后台提取触发条件：**

| 功能 | 触发条件 | 后台方式 |
|------|---------|---------|
| 记忆 | `update_memory_on_run=True` 且 `not enable_agentic_memory` | `background_executor.submit()` |
| 学习 | `agent._learning is not None` | `background_executor.submit()` |
| 文化 | `update_cultural_knowledge=True` | `background_executor.submit()` |

> 注意：`enable_agentic_memory=True` 时不走后台提取，而是通过工具由模型主动调用。

### tool_hooks 执行链速查（tools/function.py）

| 函数 | 行号 | 说明 |
|------|------|------|
| `Function.tool_hooks` | L168 | hook 列表属性 |
| `_build_hook_args()` | L898 | hook 参数注入（检查签名自动注入 agent/run_context/arguments 等） |
| `_build_nested_execution_chain()` | L928 | 构建嵌套调用链（reduce 包裹 hooks） |
| 有 hook 时执行 | L1007-1009 | 执行链式调用，hook 可拦截并返回结果 |

**tool_hooks 参数注入映射（_build_hook_args L904-926）：**

| hook 函数参数名 | 注入内容 | 说明 |
|----------------|---------|------|
| `agent` | `self.function._agent` | Agent 实例 |
| `team` | `self.function._team` | Team 实例 |
| `run_context` | `self.function._run_context` | RunContext（含 session_state） |
| `name` / `function_name` | 工具函数名 | |
| `function` / `func` / `function_call` | next_func 回调 | 调用以继续链 |
| `args` / `arguments` | 工具调用参数 | 模型传入的参数 |

### Session Summary 速查（session/summary.py）

| 类/函数 | 行号 | 说明 |
|---------|------|------|
| `SessionSummary` | L22 | 摘要 dataclass（summary, topics, updated_at） |
| `SessionSummaryResponse` | L45 | 结构化输出 Pydantic 模型 |
| `SessionSummaryManager` | L62 | 摘要管理器（model, session_summary_prompt） |
| `get_system_message()` | L92 | 构建摘要生成的 system prompt |

### 数据库速查

| 类 | 文件 | 说明 |
|----|------|------|
| `SqliteDb` | `db/sqlite/` | SQLite 同步后端 |
| `AsyncSqliteDb` | `db/sqlite/` | SQLite 异步后端 |
| `PostgresDb` | `db/postgres/` | PostgreSQL 后端 |
| `InMemoryDb` | `db/in_memory/in_memory_db.py:27` | 内存数据库（不持久化） |

### Guardrails / Hooks 速查

> Agent 的 `pre_hooks`/`post_hooks` 统一处理护栏（Guardrail）、评估（Eval）和普通函数 hook。

#### 核心类型（guardrails/）

| 类 | 文件 | 说明 |
|----|------|------|
| `BaseGuardrail` | `guardrails/base.py` L8 | 护栏抽象基类，定义 `check()` / `async_check()` 接口 |
| `PIIDetectionGuardrail` | `guardrails/pii.py` L10 | PII 检测（SSN/信用卡/邮箱/电话，支持 `mask_pii` 脱敏模式） |
| `OpenAIModerationGuardrail` | `guardrails/openai.py` L12 | OpenAI Moderation API 审核（支持 `raise_for_categories` 自定义类别） |
| `PromptInjectionGuardrail` | `guardrails/prompt_injection.py` L9 | Prompt 注入检测（17 个默认关键词，支持 `injection_patterns` 自定义） |

#### 异常类型（exceptions.py）

| 类 | 行号 | 说明 |
|----|------|------|
| `CheckTrigger` | L122 | 枚举：`OFF_TOPIC`, `INPUT_NOT_ALLOWED`, `OUTPUT_NOT_ALLOWED`, `VALIDATION_FAILED`, `PROMPT_INJECTION`, `PII_DETECTED` |
| `InputCheckError` | L134 | 输入检查异常（携带 `message`, `check_trigger`, `additional_data`） |
| `OutputCheckError` | L155 | 输出检查异常（结构与 InputCheckError 相同） |

#### Hook 规范化（utils/hooks.py）

| 函数 | 行号 | 说明 |
|------|------|------|
| `normalize_pre_hooks()` | L70 | 将 `BaseGuardrail` → `.check`/`.async_check` 绑定方法，`BaseEval` → `.pre_check`，普通函数保留 |
| `normalize_post_hooks()` | L113 | 同上，`BaseEval` → `.post_check` |
| `filter_hook_args()` | L156 | 根据函数签名过滤参数（`inspect.signature` → 仅传递接受的参数） |
| `is_guardrail_hook()` | L57 | 判断 hook 是否为护栏（后台模式下护栏必须同步执行） |
| `should_run_hook_in_background()` | L42 | 检查 `@hook(run_in_background=True)` 装饰器 |
| `copy_args_for_background()` | L15 | 深拷贝敏感参数（run_input/run_context 等）防止竞态 |

#### Hook 执行（agent/_hooks.py）

| 函数 | 行号 | 说明 |
|------|------|------|
| `execute_pre_hooks()` | L43 | 同步版 pre_hooks 执行（按顺序，`InputCheckError` 直接传播） |
| `aexecute_pre_hooks()` | L156 | 异步版 pre_hooks 执行 |
| `execute_post_hooks()` | L266 | 同步版 post_hooks 执行（参数为 `run_output` 而非 `run_input`） |
| `aexecute_post_hooks()` | L369 | 异步版 post_hooks 执行 |

**pre_hooks 参数注入映射（_hooks.py L62-70）：**

| hook 函数参数名 | 注入内容 | 说明 |
|----------------|---------|------|
| `run_input` | `RunInput` 实例 | 用户输入（可修改，如 PII 脱敏） |
| `agent` | `Agent` 实例 | |
| `session` | `AgentSession` | 会话对象 |
| `run_context` | `RunContext` | 运行上下文 |
| `user_id` | 用户 ID | |
| `debug_mode` | 调试模式 | |
| `metadata` | `run_context.metadata` | |

**post_hooks 参数注入映射（_hooks.py L285-293）：**

| hook 函数参数名 | 注入内容 | 说明 |
|----------------|---------|------|
| `run_output` | `RunOutput` 实例 | 模型输出（含 content/status 等） |
| `agent` | `Agent` 实例 | |
| `session` | `AgentSession` | |
| `run_context` | `RunContext` | |
| `user_id` | 用户 ID | |

#### _run.py 中的 Hook 调用点

| 步骤 | 行号 | 说明 |
|------|------|------|
| 规范化 | L1250-1256 | 首次运行时 `normalize_pre/post_hooks()`（`_hooks_normalised` 标记） |
| 步骤 4 pre_hooks | L410-425 | 模型调用前执行 `execute_pre_hooks()` |
| 步骤 10 post_hooks | L559-572 | 模型响应后执行 `execute_post_hooks()` |
| 异常处理 | L628-646 | 捕获 `InputCheckError`/`OutputCheckError` → `RunStatus.error` |

#### 输入/输出容器（run/agent.py）

| 类 | 行号 | 说明 |
|----|------|------|
| `RunInput` | L29 | 输入容器（`input_content`, `images`, `videos`, `audios`, `files`） |
| `RunInput.input_content_string()` | L49 | 将各种输入类型转为字符串（str/BaseModel/Message/list） |
| `RunOutput` | L581 | 输出容器（`content`, `status`, `session_state`, `metrics` 等） |

**后台执行逻辑（_hooks.py L79-97）：**

当 `_run_hooks_in_background=True` 时：
1. 护栏（`is_guardrail_hook`）始终同步执行（`InputCheckError` 必须传播）
2. 非护栏 hook 缓冲，待所有护栏通过后才提交到后台任务队列
3. 深拷贝 `run_input`/`run_context` 防止竞态条件

### 其他速查

| 函数/类 | 文件 | 行号 | 说明 |
|---------|------|------|------|
| `format_message_with_state_variables()` | `agent/_messages.py` | L56 | 模板变量替换（session_state/dependencies） |
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
- **按特性决定额外源码**：扫描目标 `.py` 文件后，根据以下特征决定需要读取的额外源码：
  - `knowledge` / `Knowledge` → 读 `knowledge/knowledge.py`（L41-55 类定义 + L507-546 search + L2879-2933 build_context + L3303-3324 retrieve）
  - `search_knowledge` / `add_knowledge_to_context` → 读 `_default_tools.py` L103-282（create_knowledge_search_tool）+ `_messages.py` L905-954（Traditional RAG 注入）
  - `knowledge_retriever` → 读 `_messages.py` L1665-1761（get_relevant_docs_from_knowledge，含 retriever 优先路径 L1716-1732）
  - `knowledge_filters` / `enable_agentic_knowledge_filters` → 读 `filters.py` L46-106（FilterExpr/EQ）+ `utils/knowledge.py`（全文）+ `_default_tools.py` L103-282
  - `references_format` → 已包含在 `_default_tools.py` L118-128（`_format_results`）
  - `ReasoningTools` → 读 `tools/reasoning.py`（全文，~291 行）
  - `tools=<callable>` → 读 `utils/callables.py`
  - `Toolkit` 子类 → 读 `tools/toolkit.py`
  - `Literal[...]` 参数 → 读 `utils/json_schema.py`
  - `Team(members=...)` → 读 `team/team.py` + `utils/callables.py`
  - `tool_call_limit` / `tool_choice` → 读 `_run.py` 对应行
  - `session_state` / `resolve_in_context` → 读 `_messages.py` L56-98（`format_message_with_state_variables`）
  - `add_session_state_to_context` → 读 `_messages.py` L437-438
  - `enable_agentic_state` → 读 `_tools.py` L165-171 + `_default_tools.py` L347-380
  - `tool_hooks` → 读 `tools/function.py` L898-974（`_build_hook_args` + `_build_nested_execution_chain`）
  - `search_session_history` → 读 `_tools.py` L143-148 + `_default_tools.py` L411-480
  - `enable_session_summaries` → 读 `session/summary.py` L22-106 + `_run.py` L589
  - `add_history_to_context` → `_messages.py` L1231-1262（已包含在基础读取范围）
  - `store_history_messages` → `agent.py` L213（已包含在基础读取范围）
  - `stream_events` / `RunCompletedEvent` → 读 `run/agent.py` L134-278
  - `get_session_state` / `update_session_state` → 读 `agent.py` L939-953
  - `get_chat_history` → 读 `agent.py` L1005-1011
  - `AsyncSqliteDb` / 异步模式 → 确认使用 `aprint_response` / `asyncio.run`
  - `InMemoryDb` → 读 `db/in_memory/in_memory_db.py` L27
  - `learning` / `LearningMachine` → 读 `learn/machine.py`（L53-120 + L350-465 + L498-540）+ `learn/config.py`（全文）
  - `enable_agentic_memory` / `memory_manager` / `MemoryManager` → 读 `memory/manager.py`（L44-100 + L481-517 + L958-986）+ `_default_tools.py` L38-75 + `_messages.py` L282-320
  - `update_memory_on_run` → 读 `_managers.py` L29-50 + L177-210
  - `add_memories_to_context` → 读 `_messages.py` L282-320（步骤 3.3.9 记忆注入 + agentic 说明）
  - `pre_hooks` / `post_hooks` / `BaseGuardrail` → 读 `agent/_hooks.py`（全文，~470 行）+ `utils/hooks.py`（全文，~179 行）
  - `PIIDetectionGuardrail` → 读 `guardrails/pii.py`（全文，~95 行）
  - `OpenAIModerationGuardrail` → 读 `guardrails/openai.py`（全文，~145 行）
  - `PromptInjectionGuardrail` → 读 `guardrails/prompt_injection.py`（全文，~53 行）
  - 自定义 `BaseGuardrail` 子类 → 读 `guardrails/base.py`（全文，~20 行）+ `exceptions.py` L122-173（CheckTrigger/InputCheckError/OutputCheckError）
  - `RunInput` / `RunOutput`（hook 参数）→ 读 `run/agent.py` L29-65（RunInput）+ L581-624（RunOutput）
  - `InputCheckError` / `OutputCheckError` 处理 → 读 `_run.py` L628-646（异常捕获 → RunStatus.error）
- **用 TaskCreate 跟踪批量进度**：批量模式下为每个文件创建 task，完成后标记 completed，便于断点续做
- **`utils/callables.py` 较小（~613 行）**：可一次性全文读取，无需分段

## 错误处理

### Write/Read 工具报错

- 如果 Write 输出文档时报错，直接重试（最多 2 次）
- 如果 Read 源码时报错，直接重试（最多 2 次）
- 所有重试均为即时重试，无需等待间隔

### 关于 Agent 子进程

> **禁止使用 Agent 子进程。** 实战中 Agent 子进程多次出现 `API Error: Failed to parse JSON` 等不可恢复错误，即使重试 3 次也无法解决。直接处理（主进程 Read + Write）**零失败**，效率更高。

## 特殊文件类型处理

### 多 Agent 文件（如 tool_choice.py）

当一个文件定义多个 Agent 做对比时：
- **核心配置一览**：用对比表格，每个 Agent 一行，突出差异字段
- **完整 API 请求**：为每个 Agent 各写一个请求示例，用注释标注差异
- **Mermaid 流程图**：用条件分支（菱形节点）展示不同策略的执行路径

### Team 文件（如 03_team_callable_members.py）

当文件使用 Team 时：
- **核心配置一览**：分组列出 Team 配置和各成员 Agent 配置
- **架构分层**：展示 Team → 成员 Agent 的委派关系
- **System Prompt 组装**：注明是 Team 协调层的 prompt，各成员 Agent 有各自独立的 prompt
- **完整 API 请求**：展示 Team 协调层的请求（含 transfer_to_X 工具）和代表性成员 Agent 的请求
- **核心调用链**：使用 Team 的调用链而非 Agent 的

### Session/State 文件

当文件涉及 session_state、历史消息、会话持久化等机制时：
- **核心配置一览**：重点标注 `session_state`、`add_session_state_to_context`、`enable_agentic_state`、`add_history_to_context`、`store_history_messages`、`search_session_history`、`enable_session_summaries` 等开关
- **核心组件解析**：解释状态的生命周期（初始化 → RunContext 传递 → 工具函数修改 → 持久化）
- **System Prompt 组装**：特别关注 `{var}` 模板变量替换（步骤 fmt）和 `<session_state>` XML 注入（步骤 3.3.17）
- **对比表**：如果该文件的机制与其他文件有对比意义（如 `enable_agentic_state` vs 自定义工具），用表格对比

### Memory/Learning 文件

当文件涉及 `MemoryManager`、`enable_agentic_memory`、`LearningMachine` 等记忆/学习机制时：
- **架构分层**：如涉及双模型（主模型 + 记忆/学习模型），在底部展示两个模型节点，标注各自用途
- **核心组件解析**：区分两种记忆模式（`enable_agentic_memory` 工具式 vs `update_memory_on_run` 自动式）、学习模式（`LearningMode.AGENTIC` vs `ALWAYS`）
- **System Prompt 组装**：特别关注步骤 3.3.9（记忆注入 + `<updating_user_memories>` 工具说明）和步骤 3.3.12（学习上下文注入 `_learning.build_context()`）
- **完整 API 请求**：如涉及 MemoryManager 的独立模型调用（如 `update_memory_task` 触发 gpt-5-mini），展示主模型请求和 MemoryManager 内部请求两个
- **Mermaid 流程图**：展示工具调用触发 MemoryManager/LearningMachine 的内部流程，用 subgraph 分组

### Knowledge/RAG 文件

当文件涉及 `Knowledge`、`search_knowledge`、`add_knowledge_to_context`、`knowledge_retriever`、`knowledge_filters` 等知识库机制时：
- **核心配置一览**：重点标注 `knowledge`（向量数据库类型）、`search_knowledge`（Agentic RAG）、`add_knowledge_to_context`（Traditional RAG）、`knowledge_filters`（静态过滤）、`enable_agentic_knowledge_filters`（代理式过滤）、`knowledge_retriever`（自定义检索）、`references_format`（引用格式）
- **架构分层**：展示 Knowledge → VectorDb → Embedder/Reranker 的层次关系，Reranker 对 Agent 层透明
- **核心组件解析**：
  - 区分两种 RAG 模式：Agentic RAG（`search_knowledge=True`，工具调用）vs Traditional RAG（`add_knowledge_to_context=True`，预注入用户消息）
  - 如有 Reranker，解释两阶段检索（初始检索 → 重排序）
  - 如有 `knowledge_retriever`，解释优先级和参数注入机制
  - 如有过滤器，解释静态 vs 代理式的区别和优先级
- **System Prompt 组装**：关注步骤 3.3.13（`Knowledge.build_context()` 注入搜索指令），如 `enable_agentic_knowledge_filters=True` 会额外注入过滤器使用指令
- **完整 API 请求**：
  - Agentic RAG：展示多轮请求（工具调用 + 工具结果），注意 `search_knowledge_base` 的工具签名（有无 `filters` 参数取决于 `enable_agentic_knowledge_filters`）
  - Traditional RAG：展示单轮请求，用户消息末尾包含 `<references>` 标签
- **对比表**：如同一文件展示多种 RAG 策略（如 knowledge_filters.py 的 static vs agentic），用对比表格突出差异

### Guardrails / Hooks 文件

当文件涉及 `pre_hooks`、`post_hooks`、`BaseGuardrail`、`PIIDetectionGuardrail`、`OpenAIModerationGuardrail`、`PromptInjectionGuardrail` 等护栏/hook 机制时：
- **核心配置一览**：重点标注 `pre_hooks`（列出所有注册的 hook/guardrail）和 `post_hooks`，如混合使用多种类型需逐个列出
- **架构分层**：展示 hook 的执行位置（pre_hooks 在模型调用前，post_hooks 在模型调用后），如 `OpenAIModerationGuardrail` 涉及外部 API 调用，在底部展示额外的 Moderation API 节点
- **核心组件解析**：
  - 区分 `BaseGuardrail` 子类（规范化为 `.check`/`.async_check` 绑定方法）和普通函数 hook（直接保留）
  - 解释 `normalize_pre_hooks()` 的规范化行为和 `filter_hook_args()` 的参数注入机制
  - 如有 `mask_pii=True`，解释脱敏模式修改 `run_input.input_content` 的数据流
  - 如混合 hook 和 guardrail，说明执行顺序和异常传播区别（guardrail 的 `InputCheckError` 会传播，普通 hook 的异常仅记录日志）
- **完整 API 请求**：
  - 检查通过时：正常展示 API 请求
  - 检查失败时：明确标注「不会发出 API 请求」，描述异常传播路径和最终 `RunOutput(status=RunStatus.error)`
  - 如 `OpenAIModerationGuardrail`：先展示 Moderation API 调用，再展示主模型 API 调用
- **Mermaid 流程图**：用菱形节点展示检查通过/失败的分支，失败路径指向红色 error 节点（`fill:#ffcdd2`）
- **对比表**：如同一文件展示拦截 vs 脱敏、默认类别 vs 自定义类别等对比场景，用表格突出差异

### 异步文件

当文件使用 `aprint_response`、`AsyncSqliteDb`、`asyncio.run` 等异步模式时：
- 在架构分层中标注 `(async)`
- 在 API 请求中使用相同格式（异步只影响 agno 层，不影响 API 请求格式）
- 在 Mermaid 流程图中不需要特别区分同步/异步

## 注意事项

- **不要虚构行号**：必须通过实际 Read/Grep 确认源码行号
- **不要遗漏配置项**：核心配置一览表要列出所有显式参数
- **API 请求要准确**：OpenAIResponses 用 `input` 参数、role 映射 `system→developer`；OpenAIChat 用 `messages` 参数、role 保持 `system`
- **Mermaid 节点 ID 不能有空格**：用驼峰或下划线命名
- **跳过已存在的 .md 文件**：批量模式下，如果同名 .md 已存在则跳过
- **先扫描再读源码**：批量模式下，先读完所有 `.py` 文件识别特性，再按需读取源码，避免读取不需要的文件
