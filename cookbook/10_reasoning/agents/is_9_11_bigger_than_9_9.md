# is_9_11_bigger_than_9_9.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/agents/is_9_11_bigger_than_9_9.py`

## 概述

本示例对比 **无推理 / 内置推理 / Claude / DeepSeek 推理模型** 在 **9.11 vs 9.9** 小数比较上的表现：多 `Agent` 变体并列运行（`rich` 控制台输出）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `regular_agent_openai` | 仅 `gpt-4o` | 基线 |
| `cot_agent_openai` | `reasoning=True` | OpenAI COT |
| `regular_agent_claude` | `Claude` | 基线 |
| `deepseek_agent_claude` / `deepseek_agent_openai` | `reasoning_model=DeepSeek` | 外接推理 |

### 还原 task

```text
9.11 and 9.9 -- which is bigger?
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/anthropic` | Claude |
| `agno/models/deepseek` | DeepSeek |
