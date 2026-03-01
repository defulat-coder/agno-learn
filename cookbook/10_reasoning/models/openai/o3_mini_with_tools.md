# o3_mini_with_tools.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/openai/o3_mini_with_tools.py`

## 概述

本示例展示 **OpenAI o3-mini 原生推理模型 + `WebSearchTools`** 的组合。o3-mini 使用内置推理能力规划金融分析步骤，同时通过工具调用获取实时网络数据，实现推理能力与信息获取的结合。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="o3-mini")` | OpenAI o3-mini 原生推理模型 |
| `tools` | `[WebSearchTools(enable_news=False)]` | 网络搜索工具（禁用新闻） |
| `instructions` | `"Use tables to display data."` | 表格格式化指令 |
| `markdown` | `True` | Markdown 格式化 |

## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 3.1 | `instructions` | `"Use tables to display data."` | 是 |
| 3.2.1 | `markdown` | `True` | 是 |
| 3.3.5 | `_tool_instructions` | WebSearchTools 说明 | 是 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["agent.print_response(NVDA vs TSLA, stream=True)"] --> B["Agent._run()"]
    B --> C["Agentic Loop（o3-mini 内置推理）"]

    subgraph Loop
        C --> D["o3-mini 内置推理<br/>规划搜索策略"]
        D --> E["web_search() 获取数据"]
        E --> F["o3-mini 内置推理<br/>分析数据，决定下一步"]
        F --> G{"完成?"}
        G -->|"否"| E
        G -->|"是"| H["生成对比报告（表格）"]
    end

    H --> I["流式输出"]

    style A fill:#e1f5fe
    style I fill:#e8f5e9
    style D fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/openai/chat.py` | `OpenAIChat` | OpenAI Chat Completions 模型 |
| `agno/tools/websearch.py` | `WebSearchTools` L16 | 网络搜索工具 |
