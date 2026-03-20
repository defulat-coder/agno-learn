# image_input_file_upload.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/image_input_file_upload.py`

## 概述

**`google.generativeai.upload_file`** 上传本地图，再 `Image(content=image_file)`，`gemini-2.0-flash-exp` + WebSearch。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-2.0-flash-exp")` | |
| `tools` | `[WebSearchTools()]` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["upload_file"] --> B["【关键】Image(Gemini File 对象)"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/media/image.py` | `Image` | content 类型 |
