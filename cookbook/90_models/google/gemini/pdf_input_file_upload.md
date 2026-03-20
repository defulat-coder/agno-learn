# pdf_input_file_upload.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/pdf_input_file_upload.py`

## 概述

**genai.Client 上传 PDF**，`File(external=retrieved_file)`，Agent 带 `add_history_to_context=True` 两轮追问。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-3-flash-preview")` | |
| `add_history_to_context` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["files.upload PDF"] --> B["【关键】File(external=...)"]
    B --> C["多轮同附件会话"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/media/file.py` | `File` | external 引用 |
