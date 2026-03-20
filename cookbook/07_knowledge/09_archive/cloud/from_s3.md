# from_s3.py — 实现原理分析

> 源文件：`cookbook/07_knowledge/09_archive/cloud/from_s3.py`

## 概述

与 `from_gcs.py` 对称，使用 **`S3Content`**（bucket `agno-public`, key 菜谱 PDF），`PostgresDb` + `PgVector`，同步/异步 `insert`/`ainsert`，`Agent` 同结构 **name/description/debug_mode**，**无显式 model**。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `S3Content` | bucket + key | S3 对象 |
| `Agent.description` | `"Agno 2.0 Agent Implementation"` | system 描述段 |

## 架构分层

```
S3 → Knowledge → Agent.print_response
```

## 核心组件解析

公开 bucket 可直接演示；私有桶需 IAM/密钥。

## System Prompt 组装

`description` 原样进入 system 前部（见 `_messages.py` #3.3.1）。

### 还原后的完整 System 文本（可静态部分）

```text
Agno 2.0 Agent Implementation

<additional_information>
- Use markdown to format your answers.
</additional_information>
```

## 完整 API 请求

默认 Model。

## Mermaid 流程图

```mermaid
flowchart TD
    A["【关键】S3Content"] --> B["Knowledge"]
    B --> C["Agent"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/knowledge/remote_content/remote_content.py` | `S3Content` |
