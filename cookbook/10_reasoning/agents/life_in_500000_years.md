# life_in_500000_years.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/agents/life_in_500000_years.py`

## 概述

本示例为 **推测性写作** 任务：对比 **`reasoning=True`** 与 **`reasoning_model=DeepSeek`**，任务字符串为五万年后生活短篇。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `task` | `Write a short story about life in 500000 years` | 创意写作 |

### 还原 task

```text
Write a short story about life in 500000 years
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/models/deepseek` | 外接推理 |
