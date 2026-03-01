# basic_reasoning_stream.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/vertex_ai/basic_reasoning_stream.py`

## 概述

本示例展示通过 **Google Vertex AI 部署的 Gemini + `reasoning=True` + 异步流式推理事件**。与 `gemini/async_reasoning_stream.py` 的核心差异是加入 `vertexai=True` 参数，将 API 路由从 Google AI Studio 切换到 Google Cloud Vertex AI，适合企业级 GCP 环境。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `reasoning_model` | `Gemini(id="gemini-2.5-flash", vertexai=True, thinking_budget=1024, include_thoughts=True)` | Vertex AI 上的 Gemini 推理 |
| `reasoning` | `True` | 启用 Agno 推理包装 |
| `instructions` | `"Think step by step about the problem."` | 推理引导 |
| 执行方式 | `asyncio.run(streaming_reasoning())` | 异步执行 |

## 核心组件解析

### vertexai=True 参数的作用

```python
Gemini(
    id="gemini-2.5-flash",
    vertexai=True,       # 切换到 Vertex AI API
    thinking_budget=1024,
    include_thoughts=True,
)
```

- `vertexai=True` 将 API 端点从 `generativelanguage.googleapis.com` 切换到 `aiplatform.googleapis.com`
- 需要 Google Cloud 项目、GCP 认证（ADC 或 Service Account），而非简单的 API Key
- 数据不离开 GCP 项目，满足企业数据驻留要求

### Google AI Studio vs Vertex AI

| 特性 | AI Studio (vertexai=False) | Vertex AI (vertexai=True) |
|------|--------------------------|--------------------------|
| 认证 | `GOOGLE_API_KEY` | GCP ADC / Service Account |
| 数据处理 | Google 政策 | GCP VPC 内 |
| SLA | 无 | 企业级 SLA |
| 适用场景 | 开发/原型 | 生产/企业 |

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.1 | `instructions` | `"Think step by step about the problem."` | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `Gemini(vertexai=True)` | Vertex AI Gemini 模型 |
| `agno/agent/_response.py` | `handle_reasoning_stream()` L86 | 异步流式推理事件 |
