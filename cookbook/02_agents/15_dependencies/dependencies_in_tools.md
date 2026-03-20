# dependencies_in_tools.py — 实现原理分析

> 源文件：`cookbook/02_agents/15_dependencies/dependencies_in_tools.py`

## 概述

本示例展示 Agno 的 **RunContext.dependencies 在工具内访问** 机制：工具函数签名中的 `run_context: RunContext` 由框架注入；`agent.run(..., dependencies={...})` 在单次运行传入业务数据（含可调用 `get_current_context`），工具从 `run_context.dependencies` 读取，无需把大块数据拼进 user 消息（本例未开启 `add_dependencies_to_context`）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(id="gpt-5.2")` | Responses API |
| `tools` | `[analyze_user]` | 单工具，依赖 RunContext |
| `name` | `"User Analysis Agent"` | 若 `add_name_to_context` 未开则不进入默认名段 |
| `description` | 长字符串 | system description |
| `instructions` | 字符串列表 | system instructions |
| `add_dependencies_to_context` | `None` | 未设置，默认不把依赖拼 user |

## 架构分层

```
用户代码层                agno.agent 层
┌──────────────────────┐    ┌────────────────────────────────────────┐
│ agent.run(           │    │ 构建 RunContext.dependencies           │
│   dependencies={...})│───>│ 工具调用时注入 run_context             │
└──────────────────────┘    └────────────────────────────────────────┘
```

## 核心组件解析

### RunContext 注入

工具 `analyze_user(user_id, run_context)` 从 `run_context.dependencies` 取 `user_profile` 与 `current_context`；其中 `current_context` 为可调用，在依赖解析阶段求值（框架约定）。

### 运行机制与因果链

1. **路径**：user 请求分析 john_doe → 模型调 `analyze_user` → 工具打印并返回聚合分析字符串 → 模型生成最终答复。
2. **副作用**：`session_id` 传入；无示例 DB。
3. **分支**：若 `dependencies` 为空，工具分支返回「No data sources available」。
4. **差异**：与 `dependencies_in_context.py` 对照，本例数据**主要给工具用**，不强调拼进 user 消息。

## System Prompt 组装

本文件显式提供 `description` 与多行 `instructions`，按 `_messages.py` `# 3.3.1`–`# 3.3.3` 进入默认 system。

### 还原后的完整 System 文本

```text
An agent specialized in analyzing users using integrated data sources.

You are a user analysis expert with access to user analysis tools.
When asked to analyze any user, use the analyze_user tool.
This tool has access to user profiles and current context through integrated data sources.
After getting tool results, provide additional insights and recommendations based on the analysis.
Be thorough in your analysis and explain what the tool found.

（多行 instructions 在 use_instruction_tags 默认下可能为 `- 行` 列表格式；上列为字面量内容。）
```

## 完整 API 请求

`OpenAIResponses` + 工具：Responses API 原生工具循环。

## Mermaid 流程图

```mermaid
flowchart TD
    A["run(dependencies=...)"] --> B["RunContext 带依赖"]
    B --> C["【关键】工具从 run_context.dependencies 读数据"]
    C --> D["模型综合 tool 输出"]
```

- **【关键】工具从 run_context.dependencies 读数据**：与注入 user 上下文的示例形成对比。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/run/__init__.py` / `run_context` | `RunContext` | 依赖载体 |
| `agno/agent/_messages.py` | `get_system_message()` | description/instructions |
