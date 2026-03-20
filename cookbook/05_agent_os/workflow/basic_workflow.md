# basic_workflow.py — 实现原理分析

> 源文件：`cookbook/05_agent_os/workflow/basic_workflow.py`

## 概述

本示例展示 Agno 的 **Workflow + AgentOS 注册**：两个 `Step` 顺序执行（研究 → 内容规划），`Workflow` 绑定 `SqliteDb` 持久化会话，`AgentOS` 仅暴露工作流。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `hackernews_agent` | `OpenAIChat(gpt-5.2)`, `HackerNewsTools`, `role=...` | 研究步 |
| `content_planner` | `OpenAIChat(gpt-4o)`, `instructions` 列表 | 规划步 |
| `content_creation_workflow` | `Workflow(steps=[research_step, content_planning_step], db=SqliteDb(...))` | 主工作流 |
| `agent_os` | `workflows=[content_creation_workflow]` | 无 tracing 显式设置 |
| `description` | `"Example OS setup"` | OS 描述 |

## 架构分层

```
Workflow 引擎 → Step(Research) → Agent.run → get_system_message → OpenAIChat.invoke
            → Step(Planning)  → 同上
```

## 核心组件解析

### Step 与 Agent

每步将 `step_input` 传给绑定 `Agent`；`hackernews_agent` 用 `role` 提供角色语义（进入 `get_system_message` 的 `# 3.3.2` 若 `role` 存在）。

### 运行机制与因果链

1. **路径**：API 触发工作流 → 顺序执行两步 LLM。
2. **副作用**：`tmp/workflow.db` 会话表 `workflow_session`。
3. **定位**：最简 **线性两步** 模板。

## System Prompt 组装

分步还原。`hackernews_agent` 无 `instructions`，有 `role`：

### 还原后的完整 System 文本（hackernews_agent）

```text
<your_role>
Extract key insights and content from Hackernews posts
</your_role>
```

（另含 `# 3.2.1` markdown 若启用；本示例未设 `markdown`。）

`content_planner` 使用 `instructions` 列表，多段会按 `# 3.3.3` 拼接为多条 `- ...` 行。

## 完整 API 请求

每步一次 `chat.completions.create`（`agno/models/openai/chat.py` L412+），`model` 分别为 `gpt-5.2` 与 `gpt-4o`。

## Mermaid 流程图

```mermaid
flowchart LR
    A["Research Step"] --> B["Content Planning Step"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/workflow/workflow.py` | `Workflow` 编排 |
| `agno/agent/_messages.py` | `get_system_message()` L106+ |
| `agno/models/openai/chat.py` | `invoke()` L385+ |
