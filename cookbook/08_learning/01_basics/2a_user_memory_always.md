# 2a_user_memory_always.py — 实现原理分析

> 源文件：`cookbook/08_learning/01_basics/2a_user_memory_always.py`

## 概述

本示例展示 **`UserMemoryConfig(mode=ALWAYS)`**：非结构化观察（职业、偏好、项目等）在后台抽取并写入，与结构化 `UserProfile` 互补。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(id="gpt-5.2")` | Responses API |
| `db` | `PostgresDb(...)` | Postgres |
| `learning` | `LearningMachine(user_memory=UserMemoryConfig(mode=ALWAYS))` | 仅用户记忆 ALWAYS |
| `markdown` | `True` | 是 |

## 架构分层

`UserMemoryStore` 的 `build_context` 将条列记忆注入 system；ALWAYS 路径在 `process` 中异步抽取。

## 核心组件解析

### 与 User Profile 的分工

画像偏结构化字段；记忆偏长文本观察与偏好细节。同一用户可同时使用两示例中的机制。

### 运行机制与因果链

第二轮「推荐库」类问题可利用第一轮写入的记忆做个性化推荐。

## System Prompt 组装

静态最小块：

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

动态：`<user_memory>` 内为条列记忆或空占位。

## 完整 API 请求

```python
client.responses.create(model="gpt-5.2", input=[...])
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["长段自我介绍"] --> B["【关键】ALWAYS 记忆抽取"]
    B --> C["Postgres"]
    C --> D["后续 system 注入 memories"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/learn/stores/user_memory.py` | `build_context` / ALWAYS 抽取 |
