# structured_output.py — 实现原理分析

> 源文件：`cookbook/90_models/cohere/structured_output.py`

## 概述

**Cohere + MovieScript**，同步 `run` + `pprint`，并 **async `aprint_response`** 第二段提示。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Cohere(id="command-a-03-2025")` | Cohere |
| `description` | `"You help people write movie scripts."` | system |
| `output_schema` | `MovieScript` | 结构化 |

## System Prompt 组装

### 还原后的完整 System 文本（核心）

```text
You help people write movie scripts.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["output_schema"] --> B["【关键】chat + response_format"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/cohere/chat.py` | `get_request_params` | 结构化参数 |
