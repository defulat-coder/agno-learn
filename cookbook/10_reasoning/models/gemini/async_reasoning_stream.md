# async_reasoning_stream.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/gemini/async_reasoning_stream.py`

## 概述

本示例展示 **Gemini thinking_budget + 异步流式推理事件**。使用 `asyncio.run()` + `aprint_response()` 实现完全异步的 Gemini 推理流，是 `basic_reasoning_stream.py` 的异步版本。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `reasoning_model` | `Gemini(id="gemini-2.5-flash", thinking_budget=1024, include_thoughts=True)` | Gemini 思考模式 |
| `reasoning` | `True` | 启用 Agno 推理包装 |
| `instructions` | `"Think step by step about the problem."` | 推理引导 |
| 执行方式 | `asyncio.run(streaming_reasoning())` | 异步执行 |

## 核心组件解析

### 异步执行 vs 同步执行

| 特性 | 同步版 (basic_reasoning_stream.py) | 异步版 (async_reasoning_stream.py) |
|------|----------------------------------|----------------------------------|
| 调用方法 | `agent.print_response()` | `await agent.aprint_response()` |
| 适用场景 | 脚本、CLI | FastAPI、异步框架 |
| 事件迭代 | 同步 for | async for |

### Gemini 异步 API 调用

Gemini 的 Python SDK 支持异步，Agno 的 `Gemini` 模型类同时提供同步和异步 invoke 方法。

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.1 | `instructions` | `"Think step by step about the problem."` | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `Gemini` | Google Gemini 模型 |
| `agno/agent/agent.py` | `aprint_response()` | 异步打印响应 |
