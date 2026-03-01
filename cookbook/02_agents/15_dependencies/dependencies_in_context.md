# dependencies_in_context.py — 实现原理分析

> 源文件：`cookbook/02_agents/15_dependencies/dependencies_in_context.py`

## 概述

本示例展示 **dependencies 注入到 context** 的模式：通过 `dependencies={"key": callable}` 将函数作为依赖注入 Agent，配合 `add_dependencies_to_context=True` 将依赖函数的**返回值**自动添加到用户消息中，让 LLM 直接看到实时数据。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(gpt-5.2)` | Responses API |
| `dependencies` | `{"top_hackernews_stories": get_top_hackernews_stories}` | 依赖注入 |
| `add_dependencies_to_context` | `True` | 将依赖结果添加到用户消息 |
| `markdown` | `True` | Markdown 输出 |

## 依赖注入机制

```python
def get_top_hackernews_stories(num_stories: int = 5) -> str:
    """从 HackerNews API 实时获取热门文章"""
    stories = [...]
    return json.dumps(stories, indent=4)

agent = Agent(
    model=OpenAIResponses(id="gpt-5.2"),
    dependencies={"top_hackernews_stories": get_top_hackernews_stories},
    add_dependencies_to_context=True,  # 自动执行依赖函数并附加结果
    markdown=True,
)
```

## add_dependencies_to_context 工作机制

```
agent.run(user_query)
  ↓
执行 dependencies 中每个 callable()
  ↓
将返回值以结构化方式附加到用户消息（XML 或 JSON 格式）:
  <context>
    <top_hackernews_stories>[{...}, {...}]</top_hackernews_stories>
  </context>
  ↓
LLM 看到：用户问题 + 实时 HackerNews 数据
```

## dependencies 在上下文 vs 工具中的区别

| 使用方式 | 配置 | LLM 可见 | 适用场景 |
|---------|------|---------|---------|
| `add_dependencies_to_context=True` | 本例 | 直接看到返回值 | 静态背景数据注入 |
| 工具中通过 `run_context.dependencies` 访问 | `dependencies_in_tools.py` | 不直接可见，工具内部使用 | 工具需要依赖数据时 |

## System Prompt 组装

无自定义 instructions，仅有 dependencies 数据附加到用户消息。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `Agent(dependencies=..., add_dependencies_to_context=True)` | 依赖注入配置 |
| `agno/agent/agent.py` | run 时 context 组装逻辑 | 执行依赖 callable 并格式化 |
