# 01_always_extraction.py — 实现原理分析

> 源文件：`cookbook/08_learning/02_user_profile/01_always_extraction.py`

## 概述

本示例为 **User Profile ALWAYS** 的深入版：多会话、多 `session_id` 逐步丰富同一 `user_id` 的画像，展示渐进式抽取。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `learning` | `LearningMachine(user_profile=UserProfileConfig(mode=ALWAYS))` | 无自定义 schema |
| `instructions` | 未设置 | 未设置 |
| `model` / `db` / `markdown` | `OpenAIResponses`、`PostgresDb`、`True` | — |

## 核心组件解析

与 `01_basics/1a_user_profile_always.py` 机制相同，区别在演示脚本轮次更多（介绍→工作→偏好→昵称）。

### 运行机制与因果链

同一 `user_id` 下 `conv_1`…`conv_4` 模拟长期关系；每轮后 `user_profile_store.print` 展示累积字段。

## System Prompt 组装

无自定义 instructions，静态最小块为 markdown 附加信息；`# 3.3.12` 带画像内容随轮次增长。

```text
<additional_information>
- Use markdown to format your answers.
</additional_information>
```

## 完整 API 请求

```python
client.responses.create(model="gpt-5.2", input=[...])
```

## Mermaid 流程图

```mermaid
flowchart LR
    C1["conv_1"] --> C2["conv_2"] --> C3["conv_3"] --> C4["conv_4"]
    C4 --> P["【关键】画像合并"]
```

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/learn/stores/user_profile.py` | ALWAYS 抽取 |
