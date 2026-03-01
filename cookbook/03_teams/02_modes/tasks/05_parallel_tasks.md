# 05_parallel_tasks.py — 实现原理分析

> 源文件：`cookbook/03_teams/02_modes/tasks/05_parallel_tasks.py`

## 概述

本示例展示 Agno 的 **tasks 模式显式并行 + 最终综合**：Leader 明确指示"使用 `execute_tasks_parallel` 并行运行三个独立分析"，三名分析师（市场/技术/财务）同时运行，结果汇入统一报告。这是对 `02_parallel.py` 的更直接演示，强调"**优先并行**"的指导原则。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Industry Analysis Team"` | Team 名称 |
| `model` | `OpenAIResponses(id="gpt-5.2")` | Leader（强模型） |
| `mode` | `TeamMode.tasks` | 自主任务模式 |
| `members` | `[market_analyst, tech_analyst, financial_analyst]` | 三名分析师（gpt-5-mini） |
| `max_iterations` | `10` | 循环上限 |
| `show_members_responses` | `True` | 显示各维度分析 |

## 核心组件解析

### 并行指导原则

instructions 中明确："这些任务是独立的——使用 `execute_tasks_parallel` 并发运行"，并总结为：

```
Prefer parallel execution whenever tasks do not depend on each other.
```

这是 tasks 模式效率设计的核心原则：**能并行就并行**，只有真正有依赖关系的任务才顺序执行。

### 执行流程

1. Leader 分析请求 → 识别三个独立分析维度
2. 调用 `execute_tasks_parallel([market_task, tech_task, finance_task])`
3. 三名 gpt-5-mini 成员并发运行
4. 全部完成 → Leader（gpt-5.2）综合三方分析 → 投资者报告

## Mermaid 流程图

```mermaid
flowchart TD
    A["analysis_team.print_response('Analyze EV industry...')"] --> B["tasks Leader gpt-5.2"]
    B --> C["execute_tasks_parallel([市场,技术,财务])"]

    subgraph 并发分析（gpt-5-mini × 3）
        C --> D["Market Analyst<br/>市场趋势/竞争格局"]
        C --> E["Tech Analyst<br/>技术创新/可行性"]
        C --> F["Financial Analyst<br/>财务/投资展望"]
    end

    D --> G["mark_all_complete(电动车投资报告)"]
    E --> G
    F --> G

    style A fill:#e1f5fe
    style G fill:#e8f5e9
    style C fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/team/_default_tools.py` | `execute_tasks_parallel()` | 并行任务执行工具 |
| `agno/team/_default_tools.py` | `mark_all_complete()` | 完成标记+最终综合 |
