# retry.py — 实现原理分析

> 源文件：`cookbook/90_models/azure/retry.py`

## 概述

**AzureAIFoundry** 错误 `id` + **retries** 配置，演示 Foundry 路径上的重试。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureAIFoundry(id="azure-wrong-id", retries=3, ...)` | 重试 |

## Mermaid 流程图

```mermaid
flowchart TD
    A["HttpResponseError"] --> B["【关键】重试"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/ai_foundry.py` | `invoke` L228+ | 异常映射 |
