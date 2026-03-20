# s3_url_file_input.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/s3_url_file_input.py`

## 概述

**S3 预签名 URL** 作为 `File(url=presigned_url)`，等同外部 HTTPS 输入，`gemini-3-flash-preview`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-3-flash-preview")` | |
| `markdown` | `True` | |

## Mermaid 流程图

```mermaid
flowchart TD
    A["boto3 presigned_url"] --> B["【关键】私有对象临时 URL"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/media/file.py` | `File` | url |
