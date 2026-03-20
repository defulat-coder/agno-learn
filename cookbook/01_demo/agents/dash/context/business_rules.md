# business_rules.py — 实现原理分析

> 源文件：`cookbook/01_demo/agents/dash/context/business_rules.py`

## 概述

本文件**不创建 Agent**，负责从 `knowledge/business/*.json` **加载业务指标、规则与常见坑**，并 **`build_business_context()`** 格式化为 Markdown 字符串；**`BUSINESS_CONTEXT`** 在 `dash/agent.py` 的 **f-string instructions** 中嵌入，进入 **system** 的语义与业务段落（非运行时检索）。

**核心配置一览：** 无 Agent 构造参数；导出 **`BUSINESS_CONTEXT`** 常量。

## 架构分层

```
JSON 文件 → load_business_rules → build_business_context → BUSINESS_CONTEXT
                                                              ↓
                                                    dash Agent instructions
```

## 核心组件解析

### load_business_rules

遍历 `BUSINESS_DIR` 下 `*.json`，合并 `metrics` / `business_rules` / `common_gotchas` 键。

### build_business_context

拼出 `## METRICS`、`## BUSINESS RULES`、`## COMMON GOTCHAS` 三段（`business_rules.py` L39-L70）。

### 运行机制与因果链

1. **数据路径**：磁盘 JSON → 进程内字符串 → 随 Agent import 进入 instructions。
2. **副作用**：无 DB；仅文件读取。
3. **分支**：目录不存在则返回空结构（L23-L24）。
4. **与 semantic_model 差异**：本文件偏**业务规则**，表元数据在 `semantic_model.py`。

## System Prompt 组装

**不直接调用 `get_system_message`。** 文本作为 **`instructions` 子串** 被 Dash Agent 使用，等价于 system 的一部分。

### 还原后的完整 System 文本

无法仅从此文件还原——需与 `agent.py` 中 `instructions` 合并。`BUSINESS_CONTEXT` 全文取决于 `knowledge/business/*.json` 内容；空目录时近似空字符串。

### 段落释义（模型视角）

- 为模型提供**指标定义**与**业务约束**，减少胡写 SQL。

## 完整 API 请求

本文件不发起 API。

## Mermaid 流程图

```mermaid
flowchart LR
    A["business/*.json"] --> B["【关键】build_business_context"]
    B --> C["BUSINESS_CONTEXT"]
    C --> D["dash instructions"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `cookbook/01_demo/agents/dash/context/business_rules.py` | `build_business_context()` L39 | 生成业务上下文字符串 |
