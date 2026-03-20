# structured_output.md — 实现原理分析

> 源文件：`cookbook/90_models/llama_cpp/structured_output.py`

## 概述

**`LlamaCpp` + `MovieScript`**，`run()` 后 `pprint(content)`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `model` | `LlamaCpp(id="ggml-org/gpt-oss-20b-GGUF")` | 本地 |
| `description` | `You write movie scripts.` | 角色 |
| `output_schema` | `MovieScript` | 结构 |

### description 原样

```text
You write movie scripts.
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["output_schema"] --> B["【关键】OpenAILike 结构化"]
```

## 关键源码文件索引

| 文件 | 关键 |
|------|------|
| `agno/models/llama_cpp/llama_cpp.py` | `supports_*` 继承 OpenAILike |
