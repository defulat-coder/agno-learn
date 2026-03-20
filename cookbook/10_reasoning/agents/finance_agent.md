# finance_agent.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/agents/finance_agent.py`

## 概述

本示例在 **YFinance 工具** 上对比 **`reasoning=True`** 与 **`reasoning_model=DeepSeek`**；`cot_agent` 使用 `use_json_mode=True` 与 `instructions` 表格展示。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `tools` | `[YFinanceTools()]` | 行情数据 |
| `cot_agent.instructions` | `"Use tables to display data"` | 单字符串 |
| `deepseek_agent.instructions` | `["Use tables where possible"]` | 列表 |

### 还原 instructions

```text
Use tables to display data
```

```text
Use tables where possible
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/tools/yfinance` | 金融工具 |
