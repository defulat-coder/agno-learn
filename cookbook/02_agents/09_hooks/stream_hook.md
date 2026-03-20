# stream_hook.py — 实现原理分析

> 源文件：`cookbook/02_agents/09_hooks/stream_hook.py`

## 概述

本示例展示 **post_hook 结合 `run_context.metadata`**：`send_notification` 在响应完成后若 `metadata` 含 `email` 则调用占位 `send_email`（print 模拟），用于「流式生成结束后的用户通知」类集成。

**核心配置一览：**

| 配置项 | 值 |
|--------|-----|
| `name` | `"Financial Report Agent"` |
| `model` | `OpenAIResponses(id="gpt-5-mini")` |
| `post_hooks` | `[send_notification]` |
| `tools` | `[YFinanceTools()]` |
| `instructions` | 三条财务报告字面量 |

## 核心组件解析

### Metadata 守卫

`if run_context.metadata is None: return` 避免空引用。

### 运行机制与因果链

`aprint_response(..., metadata={"email": "test@example.com"}, stream=True)`：流结束后仍触发 **同一** post_hook，拿到完整 `run_output.content` 发邮件。

## System Prompt 组装

```text
You are a helpful financial report agent.
Generate a financial report for the given company.
Keep it short and concise.
```

## 完整 API 请求

YFinance 工具可能触发额外网络；主模型 `responses.create`（流式）。

## Mermaid 流程图

```mermaid
flowchart TD
    R["流式输出完成"] --> N["【关键】send_notification"]
    N --> E["send_email 打印"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/yfinance` | `YFinanceTools` |
