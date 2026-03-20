# 3b_session_context_planning.py — 实现原理分析

> 源文件：`cookbook/08_learning/01_basics/3b_session_context_planning.py`

## 概述

本示例展示 **`SessionContextConfig(enable_planning=True)`**：在会话摘要之外跟踪目标、步骤与完成度，适合任务型多轮对话。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `instructions` | `"Be very concise. Give brief, actionable answers."` | 可执行向短答 |
| `learning` | `LearningMachine(session_context=SessionContextConfig(enable_planning=True))` | 规划型会话上下文 |
| `model` / `db` / `markdown` | 同系列示例 | — |

## 核心组件解析

### 与 3a 的差异

| 维度 | 3a Summary | 3b Planning |
|------|--------------|-------------|
| 结构 | 轻量摘要 | 目标+步骤+进度 |
| 适用 | 闲聊/连贯讨论 | 部署步骤等任务流 |

## System Prompt 组装

静态还原 `instructions`：

```text
Be very concise. Give brief, actionable answers.
```

以及 markdown 附加块；`# 3.3.12` 注入含 planning 结构的会话状态（运行时）。

## 完整 API 请求

```python
client.responses.create(model="gpt-5.2", input=[...])
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["设定部署目标"] --> B["【关键】规划上下文"]
    B --> C["步骤1完成"]
    C --> D["更新进度与下一步"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/learn/` SessionContextConfig | `enable_planning` 行为 |
