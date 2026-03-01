# python_101_curriculum.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/agents/python_101_curriculum.py`

## 概述

本示例展示 Agno 的 **`reasoning`（内置链式思维）** 与 **`reasoning_model`（外部推理模型）** 在**课程设计**场景中的应用。两种方式让 Agent 先规划课程结构（知识点、顺序、练习），再输出完整的 Python 101 课程大纲。

**核心配置一览：**

| 配置项 | cot_agent | deepseek_agent | 说明 |
|--------|-----------|----------------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | `OpenAIChat(id="gpt-4o")` | Chat Completions API |
| `reasoning` | `True` | `False`（默认） | 内置 COT 推理 |
| `reasoning_model` | `None` | `DeepSeek(id="deepseek-reasoner")` | 外部推理模型 |
| `markdown` | `True` | `True` | Markdown 格式化 |

## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 3.2.1 | `markdown` | `True` | 是 |
| 其他 | 未设置 | — | 否 |

### 最终 System Prompt

```text
Use markdown to format your answers.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["task: Craft a curriculum for Python 101"] --> B["Agent._run()"]
    B --> C["handle_reasoning_stream()"]
    C --> D{"推理模式"}
    D -->|"reasoning=True"| E["gpt-4o 自推理"]
    D -->|"reasoning_model=DeepSeek"| F["deepseek-reasoner 推理"]
    E & F --> G["reasoning_content"]
    G --> H["gpt-4o 生成课程大纲"]
    H --> I["Markdown 输出"]

    style A fill:#e1f5fe
    style I fill:#e8f5e9
    style G fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `reasoning` L184, `reasoning_model` L185 | 推理配置 |
| `agno/agent/_response.py` | `handle_reasoning_stream()` L86 | 流式推理触发 |
