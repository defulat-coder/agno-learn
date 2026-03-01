# basic_reasoning_stream.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/azure_openai/basic_reasoning_stream.py`

## 概述

本示例展示通过 **Azure OpenAI 部署的 gpt-4.1 + `reasoning=True` + 异步流式推理事件**。使用 Azure 企业级部署的 GPT-4.1 作为推理模型，通过 `asyncio.run()` + `aprint_response()` 实现异步推理事件流。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `reasoning_model` | `AzureOpenAI(id="gpt-4.1")` | Azure 上的 GPT-4.1 推理 |
| `reasoning` | `True` | 启用 Agno 推理包装 |
| `instructions` | `"Think step by step about the problem."` | 推理引导 |
| 执行方式 | `asyncio.run(streaming_reasoning())` | 异步执行 |

## 核心组件解析

### 异步流式推理事件

```python
await agent.aprint_response(prompt, stream=True, stream_events=True)
```

可订阅以下事件（注释代码展示）：
- `RunEvent.reasoning_started` — 推理开始
- `RunEvent.reasoning_content_delta` — 推理内容增量流
- `RunEvent.reasoning_step` — 推理步骤
- `RunEvent.reasoning_completed` — 推理完成
- `RunEvent.run_content` — 最终答案内容

### 为何 reasoning_model 使用 AzureOpenAI 而非 OpenAIChat

Azure 部署的 GPT-4.1 仅通过 `AzureOpenAI` 类访问，其 API 端点和认证机制与公开 OpenAI API 不同。

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.1 | `instructions` | `"Think step by step about the problem."` | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/azure/openai_chat.py` | `AzureOpenAI` | Azure OpenAI Chat 模型 |
| `agno/agent/_response.py` | `handle_reasoning_stream()` L86 | 异步推理流式入口 |
