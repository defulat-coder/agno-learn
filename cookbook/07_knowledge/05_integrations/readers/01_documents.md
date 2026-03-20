# 01_documents.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/05_integrations/readers/01_documents.py`

## 概述

本示例展示 **多格式文档 Reader**：PDF 自动检测、`ExcelReader` 显式指定、以及 URL PDF；`Agent(OpenAIResponses)` + `search_knowledge=True` 做 **RAG 问答**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `Knowledge` | `Qdrant(hybrid, OpenAIEmbedder)` | 向量库 |
| `Agent.model` | `OpenAIResponses(id="gpt-5.2")` | Responses |
| `search_knowledge` | `True` | RAG |
| `markdown` | `True` | Markdown |

## 架构分层

```
path/url → Reader 解析 → 切块嵌入 → Qdrant
                                │
                                ▼
                         Agent.print_response
```

## 核心组件解析

### 自动 vs 显式 Reader

扩展名触发默认 Reader；Excel 等需 `reader=ExcelReader()` 精确控制。

### 运行机制与因果链

1. 多次 `ainsert` 写入同一 collection 不同 name。
2. 用户问题经检索增强后进入 `OpenAIResponses`。

## System Prompt 组装

默认；`markdown=True`。

### 还原后的完整 System 文本

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

## 完整 API 请求

`client.responses.create`，`system` 映射为 `developer`。

## Mermaid 流程图

```mermaid
flowchart TD
    A["PDF / Excel / URL"] --> B["【关键】Reader 解析"]
    B --> C["嵌入 + Qdrant"]
    C --> D["【关键】Agent RAG"]
    D --> E["OpenAIResponses"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/reader/*` | 各格式 Reader |
| `agno/agent/_messages.py` | 消息与检索 |
| `agno/models/openai/responses.py` | Responses |
