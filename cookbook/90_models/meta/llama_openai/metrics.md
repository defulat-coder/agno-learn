# metrics.md — 实现原理分析

> 源文件：`cookbook/90_models/meta/llama_openai/metrics.py`

## 概述

**流式 `agent.run(..., stream=True)` + `pprint_run_response` + `get_last_run_output()`**，并遍历 **`agent.run_response.messages`** 打印 metrics（与同步 metrics 示例不同）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LlamaOpenAI(id="Llama-4-Maverick-17B-128E-Instruct-FP8")` | OpenAI 兼容 |
| `tools` | `[YFinanceTools()]` | 工具 |
| `markdown` | `True` | Markdown |

## 运行机制与因果链

流式迭代 `RunOutputEvent` 后，用 **`get_last_run_output()`** 取聚合结果；**`get_session_metrics()`** 打印会话级指标。

## Mermaid 流程图

```mermaid
flowchart TD
    A["stream run"] --> B["【关键】last run metrics + session metrics"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/agent/agent.py` | `get_last_run_output`、`get_session_metrics` |
