# session_options.py — 实现原理分析

> 源文件：`cookbook/02_agents/05_state_and_session/session_options.py`

## 概述

**`store_history_messages=False`**：**执行时仍 `add_history_to_context=True`**（本轮可读到之前消息），但**不把历史消息持久化进 DB**，适合隐私或减小存储。

**核心配置一览：**

| 配置项 | 值 |
|--------|-----|
| `add_history_to_context` | `True` |
| `num_history_runs` | `3` |
| `store_history_messages` | `False` |

## 架构分层

```
内存中拼历史 → 模型可见 → 落库时剥离 from_history 消息
```

## 核心组件解析

第二次 run 仍答对名字与爱好，但存储中无历史副本（`session_options.py` L37-45）。

### 运行机制与因果链

**与完全关闭 history 不同**：关闭存储 ≠ 关闭上下文（在同进程/同会话内仍可能保留）。

## System Prompt 组装

无 instructions。

## 完整 API 请求

**OpenAIResponses**。

## Mermaid 流程图

```mermaid
flowchart TD
    A["add_history True"] --> B["【关键】store_history_messages False"]
    B --> C["执行知上下文，库无历史条"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `store_history_messages` | 配置 |
