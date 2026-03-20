# structured_output.py — 实现原理分析

> 源文件：`cookbook/90_models/azure/ai_foundry/structured_output.py`

## 概述

**两个 Agent**：`strict_output=True`（默认）与 **`strict_output=False`** 的 `AzureAIFoundry(gpt-4o)`，对比结构化强度。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `structured_output_agent` | `AzureAIFoundry(id="gpt-4o"), output_schema=MovieScript` | 严格 |
| `guided_output_agent` | `AzureAIFoundry(id="gpt-4o", strict_output=False), ...` | 引导模式 |
| `description` | `"You write movie scripts."` | system |

## System Prompt 组装

### 还原后的完整 System 文本（description）

```text
You write movie scripts.
```

## Mermaid 流程图

```mermaid
flowchart TD
    S["strict_output True"] --> A["强约束"]
    G["strict_output False"] --> B["【关键】引导模式可偏离"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/ai_foundry.py` | `get_request_params` | strict_output |
