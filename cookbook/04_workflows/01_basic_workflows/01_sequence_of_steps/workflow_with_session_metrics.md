# workflow_with_session_metrics.py — 实现原理分析

> 源文件：`cookbook/04_workflows/01_basic_workflows/01_sequence_of_steps/workflow_with_session_metrics.py`

## 概述

本示例展示 Agno Workflow 的 **`会话指标收集`** 机制：通过 `workflow.run()` 获取 `WorkflowRunOutput`，再调用 `workflow.get_session_metrics()` 读取聚合的 token 用量和耗时等指标。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `Workflow.name` | `"Content Creation Workflow"` | 内容创作工作流 |
| `Workflow.steps` | `[research_step, content_planning_step]` | 两步顺序 |
| 运行方式 | `workflow.run(input=...)` | 返回 `WorkflowRunOutput` |
| 指标获取 | `workflow.get_session_metrics()` | 会话级别汇总指标 |
| 输出展示 | `pprint_run_response()` + `pprint()` | 格式化打印 |

## 架构分层

```
用户代码层                           agno.workflow 层
┌──────────────────────────────┐    ┌────────────────────────────────────────┐
│ workflow_with_session_       │    │ Workflow.run()                         │
│ metrics.py                   │    │  ├─ Step(research_team)._run()         │
│                              │    │  │   └─ 记录 RunMetrics                 │
│ response = workflow.run(...)  │───>│  ├─ Step(content_planning)._run()     │
│ pprint_run_response(response)│    │  │   └─ 记录 RunMetrics                 │
│                              │    │  └─ 聚合 WorkflowRunOutput             │
│ metrics = workflow.          │    └────────────────────────────────────────┘
│   get_session_metrics()      │
│ pprint(metrics)              │    Workflow 内部维护 WorkflowSession
└──────────────────────────────┘    ├─ step_metrics: Dict[step_name, StepMetrics]
                                    └─ duration, token_usage 等
```

## 核心组件解析

### workflow.run() vs workflow.print_response()

| 方法 | 返回值 | 用途 |
|------|--------|------|
| `workflow.run()` | `WorkflowRunOutput` | 编程访问结果和指标 |
| `workflow.print_response()` | `None` | 直接打印输出 |
| `workflow.arun()` | `WorkflowRunOutput` (async) | 异步编程访问 |

### get_session_metrics()

```python
session_metrics = content_creation_workflow.get_session_metrics()
# 返回 SessionMetrics，包含：
# - 每个步骤的 RunMetrics（token 用量、耗时）
# - 总 token 用量
# - 总执行时间
pprint(session_metrics)
```

### pprint_run_response()

```python
from agno.utils.pprint import pprint_run_response

response = content_creation_workflow.run(input="AI trends in 2024")
pprint_run_response(response, markdown=True)
# 输出格式化的工作流响应（各步骤的 content）
```

## 指标数据结构

```
SessionMetrics
├─ steps: Dict[str, StepMetrics]
│   ├─ "Research Step"
│   │   ├─ executor_type: "team"
│   │   ├─ executor_name: "Research Team"
│   │   └─ metrics: RunMetrics(input_tokens, output_tokens, time)
│   └─ "Content Planning Step"
│       ├─ executor_type: "agent"
│       ├─ executor_name: "Content Planner"
│       └─ metrics: RunMetrics(input_tokens, output_tokens, time)
└─ duration: float  # 总执行时间（秒）
```

## Mermaid 流程图

```mermaid
flowchart TD
    A["workflow.run(input='AI trends in 2024')"] --> B["Workflow._run()"]

    subgraph 步骤执行（自动记录指标）
        B --> C["Step: Research Step<br/>research_team (HN + Web)<br/>RunMetrics 记录"]
        C --> D["Step: Content Planning Step<br/>content_planner (gpt-4o)<br/>RunMetrics 记录"]
    end

    D --> E["WorkflowRunOutput<br/>（含各步骤 StepOutput）"]
    E --> F["pprint_run_response(response)"]

    B --> G["WorkflowSession 更新<br/>step_metrics 聚合"]
    G --> H["workflow.get_session_metrics()"]
    H --> I["SessionMetrics<br/>（token 用量 + 耗时）"]
    I --> J["pprint(metrics)"]

    style A fill:#e1f5fe
    style E fill:#e8f5e9
    style I fill:#e8f5e9
    style G fill:#fff3e0
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.run()` | 同步运行，返回 WorkflowRunOutput |
| `agno/workflow/workflow.py` | `Workflow.get_session_metrics()` | 获取会话级聚合指标 |
| `agno/models/metrics.py` | `RunMetrics`, `SessionMetrics` | 指标数据类 |
| `agno/utils/pprint.py` | `pprint_run_response()` | 格式化打印工作流响应 |
