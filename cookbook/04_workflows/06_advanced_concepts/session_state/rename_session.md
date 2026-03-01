# rename_session.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/session_state/rename_session.py`

## 概述

本示例展示 Agno Workflow **自动生成 session 名称**机制：`workflow.set_session_name(autogenerate=True)` 在 Workflow 运行完成后调用，基于对话内容自动生成有意义的 session 标题（类似 ChatGPT 对话标题），方便在多会话场景中识别和管理会话。

**核心配置一览：**

| API | 说明 |
|-----|------|
| `workflow.set_session_name(autogenerate=True)` | 基于对话内容自动生成名称 |
| `workflow.get_session_name()` | 获取当前 session 名称 |
| 需要 `Workflow.db` | Session 名称持久化到数据库 |

## 核心组件解析

### 运行 Workflow 后自动命名

```python
article_workflow = Workflow(
    description="Automated article creation from research to writing",
    steps=[article_creation_sequence],
    db=SqliteDb(db_file="tmp/workflows.db"),
    debug_mode=True,
)

# 先运行 Workflow
article_workflow.print_response(
    input="Write an article about the benefits of renewable energy",
    markdown=True,
)

# 运行完成后，基于对话内容自动生成 session 名称
article_workflow.set_session_name(autogenerate=True)

# 读取生成的名称
print(f"New session name: {article_workflow.get_session_name()}")
# 示例输出: "Renewable Energy Benefits Article"
```

### Steps 容器结构

```python
article_creation_sequence = Steps(
    name="article_creation",
    steps=[research_step, writing_step],
)

article_workflow = Workflow(
    steps=[article_creation_sequence],  # Steps 容器
)
```

## 应用场景

| 场景 | 说明 |
|------|------|
| 多对话管理 | 为每个对话自动生成有意义的标题 |
| 会话检索 | 通过名称快速识别历史会话 |
| 用户界面 | 显示格式化的会话列表（类似 ChatGPT 侧边栏） |

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.set_session_name(autogenerate)` | 自动生成 session 名称 |
| `agno/workflow/workflow.py` | `Workflow.get_session_name()` | 获取当前 session 名称 |
