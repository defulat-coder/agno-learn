# cerebras_llama_reasoning_tools.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/tools/cerebras_llama_reasoning_tools.py`

## 概述

本示例展示 **`ReasoningTools`** 与 **Cerebras**（`llama-3.3-70b`）的组合，配合 `stream_events=True` 和 `add_datetime_to_context=True`。Cerebras 的超低延迟特性使得 ReasoningTools 的多轮工具调用能够以极快的速度完成推理循环。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cerebras(id="llama-3.3-70b")` | Cerebras 超低延迟平台 |
| `tools` | `[ReasoningTools(add_instructions=True)]` | 推理工具（含说明） |
| `instructions` | 详细的问题解决指引 | 引导分步推理 |
| `add_datetime_to_context` | `True` | 注入当前时间 |
| `stream_events` | `True` | 流式事件模式 |
| `markdown` | `True` | Markdown 格式化 |

## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 3.1 | `instructions` | 详细问题解决指引 | 是 |
| 3.2.1 | `markdown` | `True` | 是 |
| 3.2.2 | `add_datetime_to_context` | `True` → 当前时间 | 是 |
| 3.3.5 | `_tool_instructions` | ReasoningTools 使用说明 | 是 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["reasoning_agent.print_response<br/>逻辑谜题 stream=True"] --> B["Agent._run()"]
    B --> C["Agentic Loop（Cerebras 超低延迟）"]

    subgraph Loop
        C --> D["think() - 毫秒级响应"]
        D --> E["think() 继续推理"]
        E --> F{"next_action?"}
        F -->|"continue"| D
        F -->|"final_answer"| G["生成解答"]
    end

    G --> H["stream_events 极速输出"]

    style A fill:#e1f5fe
    style H fill:#e8f5e9
    style D fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/tools/reasoning.py` | `ReasoningTools` L10 | 推理工具 |
| `agno/models/cerebras` | `Cerebras` | Cerebras 模型类 |
