# reasoning_model_deepseek.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/azure_ai_foundry/reasoning_model_deepseek.py`

## 概述

本示例展示 **Azure AI Foundry 上的跨模型推理**：`model=AzureAIFoundry(gpt-4o)`（响应生成）+ `reasoning_model=AzureAIFoundry(DeepSeek-R1)`（推理），通过 Azure AI Foundry 统一访问不同厂商的模型（GPT-4o 和 DeepSeek-R1），同时满足企业合规要求。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `AzureAIFoundry(id="gpt-4o")` | 响应生成（通过 Foundry） |
| `reasoning_model` | `AzureAIFoundry(id="DeepSeek-R1", azure_endpoint=..., api_key=...)` | DeepSeek-R1（单独端点） |
| `reasoning` | `True` | 启用 Agno 推理包装 |

## 核心组件解析

### Azure AI Foundry vs Azure OpenAI

| 特性 | Azure OpenAI | Azure AI Foundry |
|------|-------------|-----------------|
| 支持模型 | 仅 OpenAI 模型 | 多厂商（OpenAI、Meta、Mistral、DeepSeek 等） |
| 端点 | 固定格式 | 每个模型部署有独立端点 |
| 认证 | 统一密钥 | 每个端点可能有独立密钥 |

### DeepSeek-R1 独立端点配置

```python
AzureAIFoundry(
    id="DeepSeek-R1",
    azure_endpoint=os.getenv("AZURE_ENDPOINT"),  # DeepSeek 专属端点
    api_key=os.getenv("AZURE_API_KEY"),
)
```

DeepSeek-R1 在 Azure AI Foundry 中作为独立部署，有专属的 endpoint 和 API key，需要单独获取。

### reasoning=True 的必要性

与某些示例（仅设 `reasoning_model` 不设 `reasoning`）不同，本文件显式设置 `reasoning=True`，确保 Agno 明确启用推理模式，避免歧义。

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 所有配置 | 均未设置 | — | 否（最简配置） |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/__init__.py` | `AzureAIFoundry` | Azure AI Foundry 多模型接口 |
| `agno/agent/_response.py` | `handle_reasoning()` L70 | 两阶段推理协调 |
