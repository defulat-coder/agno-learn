# 02_local.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/05_integrations/vector_dbs/02_local.py`

## 概述

本示例展示 **本地向量库 ChromaDB 与 LanceDB**（`try/ImportError`）：无服务端时嵌入式存储；`insert` + `Agent(OpenAIResponses)` RAG。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `knowledge_chroma` | `ChromaDb(collection, path, persistent_client=True)` | 条件 |
| `knowledge_lance` | `LanceDb(uri, table_name, SearchType.hybrid)` | 条件 |
| `Agent` | `OpenAIResponses(gpt-5.2)`, `search_knowledge=True`, `markdown=True` | 各分支 |

## 架构分层

```
PDF → Chroma 或 Lance → Agent → OpenAIResponses
```

## 核心组件解析

### ImportError 分支

未安装依赖时打印提示并跳过，避免 import 失败。

### 运行机制与因果链

适合笔记本/CI 原型；与 Qdrant 服务相比 **无独立向量进程**（Chroma 可持久化目录）。

## System Prompt 组装

`markdown=True` 默认。

### 还原后的完整 System 文本

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

## 完整 API 请求

仅 LLM 为 `responses.create`；向量操作在本地库。

## Mermaid 流程图

```mermaid
flowchart TD
    A["Import chromadb / lancedb"] --> B["【关键】本地向量 insert"]
    B --> C["Agent RAG"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/vectordb/chroma` | Chroma |
| `agno/vectordb/lancedb` | Lance |
