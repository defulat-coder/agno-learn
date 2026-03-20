# lance_db_with_mistral_embedder.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/vector_dbs/lance_db_with_mistral_embedder.py`

## 概述

**`MistralEmbedder`** + **`PDFReader(chunk_size=1024)`** + **`SearchType.hybrid`**；仅 **`ainsert`**，**无 Agent 调用**（脚本止于入库）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `embedder` | Mistral | 与 OpenAI 嵌入对照 |

## 核心组件解析

异构嵌入模型需与维度、索引配置一致；本例突出 **Mistral 嵌入管线**。

## System Prompt 组装

无 Agent，无 `get_system_message`。

## 完整 API 请求

Mistral Embeddings API；无聊天。

## Mermaid 流程图

```mermaid
flowchart TD
    A["MistralEmbedder"] --> B["【关键】非 OpenAI 嵌入 + Lance hybrid"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/embedder/mistral.py` | |
