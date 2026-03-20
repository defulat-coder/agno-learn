# mcp_tools_advanced_example.py — 实现原理分析

> 源文件：`cookbook/05_agent_os/mcp_demo/mcp_tools_advanced_example.py`

## 概述

本示例展示 Agno 的 **多 MCP 源并存**：`MCPTools(transport="streamable-http", url="https://docs.agno.com/mcp")` 与 **stdio** 型 `npx @modelcontextprotocol/server-brave-search`（需 `BRAVE_API_KEY`）；单 Agent 同时挂载两类 `MCPTools`，由模型在运行中选择调用。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `agno_mcp_tools` | 远程 HTTP MCP | Agno 文档 |
| `brave_mcp_tools` | `command` + `env` | Brave 搜索 |
| `tools` | `[brave_mcp_tools, agno_mcp_tools]` | 多 MCP |
| `model` | `Claude(id="claude-sonnet-4-0")` | Anthropic |

## 运行机制与因果链

AgentOS 管理各 `MCPTools` 生命周期（文件头注释）；**勿** `reload=True` 以免 lifespan 问题。

## System Prompt 组装

无显式 `instructions`；工具定义来自两个 MCP 会话合并。

## 完整 API 请求

`Claude.invoke` + 多 MCP 工具模式。

## Mermaid 流程图

```mermaid
flowchart TD
    A["Agent"] --> B["【关键】双 MCPTools"]
    B --> C["Brave stdio MCP"]
    B --> D["Agno HTTP MCP"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/tools/mcp` | `MCPTools` | 多传输方式 |
