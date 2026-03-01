# tool_call_compression.py — 实现原理分析

> 源文件：`cookbook/02_agents/14_advanced/tool_call_compression.py`

## 概述

本示例展示 Agent 级别的**工具结果压缩**：通过在 Agent 中设置 `compress_tool_results=True`，Agno 在工具调用完成后自动压缩过长的工具返回值，减少后续模型调用的 token 消耗。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIResponses(gpt-5-mini)` | Responses API |
| `tools` | `[WebSearchTools()]` | Web 搜索工具 |
| `compress_tool_results` | `True` | 启用工具结果自动压缩 |
| `db` | `SqliteDb` | 会话持久化 |
| `description` | 竞争对手追踪 | 定义角色 |

## compress_tool_results vs CompressionManager

| 方式 | 配置 | 适用场景 |
|------|------|---------|
| `compress_tool_results=True`（简单） | Agent 直接配置 | 使用默认压缩策略 |
| `CompressionManager(compress_token_limit=..., compress_tool_call_instructions=...)` | 自定义管理器 | 精细控制阈值和压缩提示词 |

`compress_tool_results=True` 是便捷开关，内部使用默认的 `CompressionManager` 配置。

## 核心代码模式

```python
agent = Agent(
    model=OpenAIResponses(id="gpt-5-mini"),
    tools=[WebSearchTools()],
    description="Specialized in tracking competitor activities",
    instructions="Use the search tools and always use the latest information and data.",
    db=SqliteDb(db_file="tmp/dbs/tool_call_compression.db"),
    compress_tool_results=True,  # 简单开关
)

agent.print_response(
    "Research recent activities (last 3 months) for OpenAI, Anthropic, DeepMind, Meta AI...",
    stream=True,
)
```

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.3.1 | `description` | "Specialized in tracking competitor activities" | 是 |
| 3.3.2 | `instructions` | "Use the search tools..." | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `Agent(compress_tool_results=True)` | 简单压缩开关 |
| `agno/compression/manager.py` | `CompressionManager` | 底层压缩实现 |
