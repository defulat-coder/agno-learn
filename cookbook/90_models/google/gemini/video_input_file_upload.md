# video_input_file_upload.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/video_input_file_upload.py`

## 概述

**大文件上传流程**：`files.upload` 等待 `PROCESSING`，`Video(content=video_file)`；`if __name__` 内 `print_response`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-3-flash-preview")` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["upload 视频"] --> B["【关键】File API + Video"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `get_client().files` | |
