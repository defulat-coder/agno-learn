# capture_reasoning_content_default_COT.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/agents/capture_reasoning_content_default_COT.py`

## 概述

本示例演示如何检查 **`RunOutput.reasoning_content`**：非流式与流式、`reasoning=True`，并打印长度与预览（前 1000 字符）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `agent` | `reasoning=True`，`gpt-4o` | 默认 COT |
| 辅助函数 | `print_reasoning_content` | 调试输出 |

## 核心组件解析

用于验证推理文本是否在流/非流路径上均正确填充；具体字段由模型适配器写入 `RunOutput`。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/run/agent.py` | `RunOutput` |
| `agno/models/openai` | 推理字段解析 |
