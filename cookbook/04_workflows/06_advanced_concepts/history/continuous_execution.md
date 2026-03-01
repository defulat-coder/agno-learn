# continuous_execution.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/history/continuous_execution.py`

## 概述

本示例展示 Agno Workflow **基于历史的持续对话执行模式**：单步 Workflow + `add_workflow_history_to_steps=True` + `workflow.cli_app()` 实现交互式 CLI，Agent 在每次执行时自动获取完整对话历史，适用于需要多轮上下文记忆的辅导、客服等场景。

**核心配置：**

| 配置 | 值 | 说明 |
|------|------|------|
| `add_workflow_history_to_steps` | `True` | 全局向所有 Step 注入历史 |
| `db` | `SqliteDb` | 持久化存储对话历史 |
| `workflow.cli_app()` | 交互式 CLI | 连续多轮输入 |

## 核心组件解析

### 持续对话 Workflow

```python
tutor_workflow = Workflow(
    name="Simple AI Tutor",
    db=SqliteDb(db_file="tmp/simple_tutor_workflow.db"),  # 持久化
    steps=[Step(name="AI Tutoring", agent=tutor_agent)],
    add_workflow_history_to_steps=True,  # Agent 获取完整历史
)

# 启动交互式 CLI（持续多轮对话）
tutor_workflow.cli_app(
    session_id="simple_tutor_demo",
    user="Student",
    stream=True,
    show_step_details=True,
)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.add_workflow_history_to_steps` | 历史注入 |
| `agno/workflow/workflow.py` | `Workflow.cli_app()` | 交互式 CLI 循环 |
