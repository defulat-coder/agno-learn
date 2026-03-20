# pdf_input_url.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/pdf_input_url.py`

## 概述

**HTTPS PDF URL + InMemoryDb**：`File(url=...)`，两轮对话，`add_history_to_context=True`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-3-flash-preview")` | |
| `db` | `InMemoryDb()` | 内存会话 |
| `add_history_to_context` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["PDF URL"] --> B["【关键】远程 PDF + 会话追问"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/db/in_memory.py` | `InMemoryDb` | |
