# 03_managed.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/05_integrations/vector_dbs/03_managed.py`

## 概述

本示例展示 **Pinecone 托管向量库**（`try/ImportError`）：`PINECONE_API_KEY` + `PineconeDb`，`insert` PDF 后 `Agent(OpenAIResponses)` RAG。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `PineconeDb` | `name`, `api_key`, `embedder` | 托管库 |
| `Agent` | `OpenAIResponses(gpt-5.2)`, `search_knowledge=True`, `markdown=True` | 条件执行 |

## 架构分层

```
Pinecone ← Knowledge.insert ← Agent → OpenAI
```

## 核心组件解析

未安装 `pinecone` 时跳过 demo，打印安装提示。

### 运行机制与因果链

云上命名空间/索引由 Pinecone 控制台与 SDK 管理；与自托管 Qdrant 运维模型不同。

## System Prompt 组装

`markdown=True`。

### 还原后的完整 System 文本

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

## 完整 API 请求

`responses.create`；向量请求走 Pinecone HTTP API。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】PineconeDb insert"] --> B["Agent RAG"]
    B --> C["OpenAIResponses"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/vectordb/pineconedb` | Pinecone 适配 |
