# demo_mistral.py — 实现原理分析

> 源文件：`cookbook/90_models/azure/ai_foundry/demo_mistral.py`

## 概述

**AzureAIFoundry(id="Mistral-Large-2411")**，演示 Foundry 上 Mistral 部署。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureAIFoundry(id="Mistral-Large-2411")` | Mistral |
| `markdown` | `True` | Markdown |

## System Prompt 组装

### 还原后的完整 System 文本

```text
Use markdown to format your answers.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["Mistral 部署"] --> B["complete"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/ai_foundry.py` | `invoke()` | 同上 |
