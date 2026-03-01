# dependencies_in_tools.py — 实现原理分析

> 源文件：`cookbook/02_agents/15_dependencies/dependencies_in_tools.py`

## 概述

本示例展示 **dependencies 在工具中的访问**模式：工具函数通过参数 `run_context: RunContext` 访问运行时注入的 `dependencies` 字典，可在工具内部使用业务数据（如用户画像、当前上下文）而不暴露给 LLM。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(gpt-5.2)` | Responses API |
| `tools` | `[analyze_user]` | 接受 RunContext 的工具函数 |
| `name` | `"User Analysis Agent"` | Agent 名称 |

## 核心模式：RunContext 在工具中

```python
def analyze_user(user_id: str, run_context: RunContext) -> str:
    """工具函数通过 run_context 参数自动接收运行时上下文"""
    dependencies = run_context.dependencies  # 获取注入的依赖字典
    
    if "user_profile" in dependencies:
        profile_data = dependencies["user_profile"]  # 直接数据
    
    if "current_context" in dependencies:
        context_data = dependencies["current_context"]  # 可以是 callable，已被自动调用
    
    return "\n\n".join(results)
```

## run() 时注入依赖

```python
response = agent.run(
    input="Please analyze user 'john_doe'...",
    dependencies={
        "user_profile": {          # 直接数据（dict）
            "name": "John Doe",
            "role": "Senior Software Engineer",
        },
        "current_context": get_current_context,  # callable，运行时自动调用
    },
    session_id="test_tool_dependencies",
)
```

## dependencies 值类型支持

| 值类型 | 处理方式 | 工具中访问 |
|--------|---------|---------|
| 直接值（dict/str/etc.） | 直接传入 | `run_context.dependencies["key"]` |
| callable | 运行时自动调用获取返回值 | `run_context.dependencies["key"]`（已解析） |

## RunContext 字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `dependencies` | `dict` | 注入的依赖数据（callable 已解析） |
| `session_state` | `dict` | 会话状态 |
| `run_id` | `str` | 当前 run ID |

## System Prompt 组装

```text
Your name is: User Analysis Agent

An agent specialized in analyzing users using integrated data sources.

You are a user analysis expert with access to user analysis tools.
When asked to analyze any user, use the analyze_user tool.
...
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/run/__init__.py` | `RunContext` | 工具上下文对象 |
| `agno/agent/agent.py` | `run(dependencies=...)` | 运行时依赖注入 |
| `agno/tools/toolkit.py` | RunContext 参数注入机制 | 自动填充工具的 run_context 参数 |
