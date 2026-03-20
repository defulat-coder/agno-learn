# sequential_workflow.py — 实现原理分析

> 源文件：`cookbook/00_quickstart/sequential_workflow.py`

## 概述

本示例展示 Agno 的 **`Workflow` + `Step`**：**顺序管道** Data → Analysis → Report；每步绑定一个 **`Agent`**，由工作流引擎按序执行，区别于 **`Team`** 的动态协作。提示词分布在**各 Step 的 Agent `instructions`**，无单一全局 Agent。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `Workflow` | `name`, `description`, `steps=[data_step, analysis_step, report_step]` | 三步管道 |
| `Step` | `name`, `agent`, `description` | 每步包装一个 Agent |
| `data_agent` / `analyst_agent` / `report_agent` | 各 `Gemini` + `instructions` + `db` | 专用子代理 |
| `workflow_db` | `SqliteDb(...)` | 共享 |

## 架构分层

```
用户输入 → Workflow.print_response
        → Step1 data_agent.run
        → Step2 analyst_agent.run（输入依赖上步输出）
        → Step3 report_agent.run
        → 最终输出
```

## 核心组件解析

### Workflow 与 Step

`Workflow` 见 `agno/workflow/workflow.py` 约 **L208**；`Step` 将 Agent 与描述绑定，引擎负责顺序与数据传递（详见 workflow 模块内 `print_response`/`run` 实现）。

### 三角色分工

- **Data Gatherer**：只拉数不解读。
- **Analyst**：解读数据。
- **Report Writer**：写简短投资建议（`markdown=True`）。

### 运行机制与因果链

1. **路径**：单用户请求触发多步串联；每步一次（或多次）子 Agent run。
2. **副作用**：各 Agent 共享 `workflow_db` 会话策略。
3. **分支**：与 Team 不同，**不**由 Leader 动态选人，顺序固定。
4. **定位**：演示 **ETL 式**可控流水线。

## System Prompt 组装

**分属三个 Agent**，各自 `get_system_message`。示例 **Data Gatherer** 还原：

```text
You are a data gathering agent. Your job is to fetch comprehensive market data.

For the requested stock, gather:
- Current price and daily change
- Market cap and volume
- P/E ratio, EPS, and other key ratios
- 52-week high and low
- Recent price trends

Present the raw data clearly. Don't analyze — just gather and organize.

Use markdown to format your answers.  # 若该 Agent markdown 默认

<additional_information>
- The current time is <运行时时间>.
</additional_information>
```

（`data_agent` 未设 `markdown=True`，故可能无 #3.2.1 行——以 `_messages.py` 条件为准。）

Analyst、Report Writer 见源码各自 `instructions` 块。

## 完整 API 请求

三步各调用 `Gemini`；前一步输出作为后一步上下文的一部分（具体字段由 Workflow 实现组装）。

## Mermaid 流程图

```mermaid
flowchart LR
    A["用户主题"] --> B["【关键】Step1 Data"]
    B --> C["【关键】Step2 Analysis"]
    C --> D["Step3 Report"]
```

- **【关键】Step1/2**：顺序与职责划分是 Workflow 的核心。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow` L208+ | 工作流执行 |
| `agno/agent/_messages.py` | `get_system_message()` | 每步 Agent 的 system |
