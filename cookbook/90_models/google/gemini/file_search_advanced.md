# file_search_advanced.py — 实现原理分析

> 源文件：`cookbook/90_models/google/gemini/file_search_advanced.py`

## 概述

**Gemini File Search**：多 store、自定义 chunking/metadata、`file_search_metadata_filter`，`agent.run` 查询并解析 `run.citations.raw` 中 grounding。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Gemini(id="gemini-2.5-flash")` | 含 `create_file_search_store`、`upload_to_file_search_store` 等 |
| `agent` | `Agent(model=model, markdown=True)` | |

## 运行机制与因果链

1. 创建两个 store，上传技术/营销文档并等待 operation。
2. 设置 `model.file_search_store_names` 与 `model.file_search_metadata_filter` 后调用 `agent.run`。
3. 从 `citations.raw["grounding_metadata"]` 提取引用。

## 完整 API 请求

底层为 `generate_content`，并带 File Search 工具/配置（见 `Gemini.get_request_params` 与 google-genai）。

## Mermaid 流程图

```mermaid
flowchart TD
    subgraph K["【关键】File Search 多 store + 元数据过滤"]
        A1["upload + chunking"] --> A2["file_search_store_names"]
        A2 --> A3["generate_content + grounding"]
    end
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/google/gemini.py` | `create_file_search_store` 等 | 扩展方法 |
| `agno/run/agent.py` | `RunOutput.citations` | 引用 |
