# skip_if_exists_contentsdb.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/lifecycle/skip_if_exists_contentsdb.py`

## 概述

本示例演示：先 **仅向量路径** 插入远程 PDF；再运行时挂上 **`PostgresDb` 作为 `contents_db`**，在 **`skip_if_exists=True`** 下再次插入，用于展示「向量已存在时如何将内容纳入 contents DB」的典型集成步骤。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 首次 `Knowledge` | 仅 `PgVector` | 无 contents_db |
| 运行时赋值 | `knowledge.contents_db = PostgresDb(...)` | 延迟挂载 |
| `skip_if_exists` | `True` | 第二次插入行为 |
| `Agent` | 无 | |

## 核心组件解析

### contents_db 延迟绑定

先写向量再补内容库，在迁移或增量同步场景常见；具体是否仅补元数据、是否触发向量重建取决于 `Knowledge.insert` 实现。

### 运行机制与因果链

1. **路径**：两次 `insert` 同一 URL，第二次在已有向量 + 新 contents_db 条件下执行。
2. **副作用**：PostgreSQL 多表协同。

## System Prompt 组装

无 Agent。

## 完整 API 请求

无 LLM。

## Mermaid 流程图

```mermaid
flowchart TD
    A[insert 无 contents_db] --> B["【关键】挂载 contents_db"]
    B --> C[skip_if_exists=True 再 insert]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/knowledge.py` | `contents_db` 与 `skip_if_exists` 协同 |
