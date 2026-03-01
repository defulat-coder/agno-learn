# reasoning_model_stream_deepseek.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/azure_ai_foundry/reasoning_model_stream_deepseek.py`

## 概述

本示例是 `reasoning_model_deepseek.py` 的**异步流式版本**：使用 Azure AI Foundry 上的 DeepSeek-R1 作为推理模型，通过 `asyncio.run()` + `aprint_response()` 实现异步流式推理事件输出，支持实时观察 DeepSeek-R1 的推理过程。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `reasoning_model` | `AzureAIFoundry(id="DeepSeek-R1", azure_endpoint=..., api_key=...)` | DeepSeek-R1（Foundry） |
| `reasoning` | `True` | 启用 Agno 推理包装 |
| `instructions` | `"Think step by step about the problem."` | 推理引导 |
| 执行方式 | `asyncio.run(streaming_reasoning())` | 异步执行 |

## 核心组件解析

### 与 reasoning_model_deepseek.py 的差异

| 特性 | reasoning_model_deepseek.py | reasoning_model_stream_deepseek.py |
|------|---------------------------|----------------------------------|
| model | gpt-4o（生成） + DeepSeek-R1（推理） | 仅 DeepSeek-R1（推理）+ 无主模型 |
| 执行模式 | 同步 | 异步（asyncio） |
| 流式事件 | 否 | 是（stream_events=True） |
| 可见推理过程 | 否 | 是（reasoning_content_delta） |

### 无主 model 的配置

本文件仅设置 `reasoning_model` 而未设置 `model`，这意味着 Agno 将直接使用推理模型的输出作为最终响应（单模型推理流），无需第二次 API 调用。

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.1 | `instructions` | `"Think step by step about the problem."` | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/__init__.py` | `AzureAIFoundry` | Azure AI Foundry 接口 |
| `agno/agent/_response.py` | `handle_reasoning_stream()` L86 | 异步流式推理事件 |
