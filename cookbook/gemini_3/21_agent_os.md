# 21_agent_os.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Agent OS - Deploy All Agents as a Web Service
===============================================
Deploy all agents, teams, and workflows from this guide as a single web service.

How to use:
1. Start the server: python cookbook/gemini_3/21_agent_os.py
2. Visit https://os.agno.com in your browser
3. Add your local endpoint: http://localhost:7777
4. Select any agent, team, or workflow and start chatting

Key concepts:
- AgentOS: Wraps agents, teams, and workflows into a FastAPI web service
- get_app(): Returns a FastAPI app you can customize
- serve(): Starts the server with uvicorn (hot-reload enabled)
- tracing=True: Enables request tracing in the Agent OS UI

Prerequisites:
- GOOGLE_API_KEY environment variable set
"""

import importlib
import sys
from pathlib import Path

from agno.os import AgentOS
from db import gemini_agents_db

# Numbered filenames need importlib since Python can't import modules starting with digits
sys.path.insert(0, str(Path(__file__).parent))


def _import(module_name: str, attr: str):
    return getattr(importlib.import_module(module_name), attr)


chat_agent = _import("1_basic", "chat_agent")
finance_agent = _import("2_tools", "finance_agent")
critic_agent = _import("3_structured_output", "critic_agent")
news_agent = _import("4_search", "news_agent")
url_agent = _import("6_url_context", "url_agent")
image_agent = _import("8_image_input", "image_agent")
doc_reader = _import("13_pdf_input", "doc_reader")
recipe_agent = _import("17_knowledge", "recipe_agent")
tutor_agent = _import("18_memory", "tutor_agent")
content_team = _import("19_team", "content_team")
research_pipeline = _import("20_workflow", "research_pipeline")

agent_os = AgentOS(
    id="gemini-agent-os",
    agents=[
        chat_agent,
        finance_agent,
        critic_agent,
        news_agent,
        url_agent,
        image_agent,
        doc_reader,
        recipe_agent,
        tutor_agent,
    ],
    teams=[content_team],
    workflows=[research_pipeline],
    db=gemini_agents_db,
    tracing=True,
)
app = agent_os.get_app()

if __name__ == "__main__":
    agent_os.serve(app="21_agent_os:app", reload=True)
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/gemini_3/21_agent_os.py`

## 概述

Agent OS - Deploy All Agents as a Web Service

本示例归类：**脚本/工具入口**；模型相关类型：`（见源码 import）`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| （见源码） | — | 请展开 `Agent` / `Team` 构造参数 |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ 21_agent_os.py       │  ──▶  │ Agent → get_run_messages → Model │
└──────────────────────┘         └────────────────────────────────┘
                                          │
                                          ▼
                                  ┌───────────────┐
                                  │ 对应 Model 子类 │
                                  └───────────────┘
```

## 核心组件解析

### 运行机制与因果链

1. **入口**：从模块 `__main__` 或暴露的 `agent` / `team` 调用进入；同步用 `print_response` / `run`，异步用 `aprint_response` / `arun`（若源码中有）。
2. **消息**：默认路径下 system 内容由 `get_system_message()`（`libs/agno/agno/agent/_messages.py` 约 **L106** 起）按分段逻辑拼装；若显式传入 `system_message` 则早退使用该字符串。
3. **模型**：具体 HTTP/SDK 形态以 `libs/agno/agno/models/` 下对应类的 `invoke` / `ainvoke` 为准（勿默认写成单一 `chat.completions`）。
4. **副作用**：若配置 `db`、`knowledge`、`memory`，运行会读写存储；仅以本文件为准对照。

### 与框架的衔接

- **System**：`get_system_message()` 锚点 `agno/agent/_messages.py` **L106+**。
- **运行**：`Agent.print_response` 等入口 `agno/agent/agent.py`（以当前仓库检索为准）。

## System Prompt 组装

| 序号 | 组成部分 | 本文件 | 是否生效 |
|------|---------|--------|---------|
| 1 | `instructions` / `description` 等 | 见核心配置表与源码 | 有赋值则生效 |
| 2 | 默认分段（markdown、时间等） | 取决于 `Agent` 默认与显式参数 | 视参数 |

### 拼装顺序与源码锚点

1. `system_message` 直给 → 使用该内容（见 `_messages.py` 文档字符串分支说明）。
2. 否则默认拼装：`description`、`role`、`instructions`、markdown 附加段等按 `# 3.x` 注释顺序合并。

### 还原后的完整 System 文本

```text
（本文件未出现 `Agent(...)` 构造；可能为脚本、工具封装或 MCP 服务，详见源码逻辑。）
```

### 段落释义（模型视角）

- 指令与安全边界由 `instructions` / `system_message` 约束；若带 `tools` / `knowledge`，文档中需体现「何时检索/调用」由框架注入的提示段支持。

## 完整 API 请求

```python
# 请以本文件实际 Model 为准打开 libs/agno/agno/models/<厂商>/ 下对应类的 invoke：
# 可能是 chat.completions.create、responses.create、Gemini generate_content 等。
```

> 与上一节 system 文本在同一 run 中组合；`developer`/`system` 角色由适配器转换。

```mermaid
flowchart TD
    Entry["用户入口<br/>`if __name__` / main"] --> Run["【关键】Agent.run / print_response"]
    Run --> Sys["【关键】get_system_message<br/>agno/agent/_messages.py L106+"]
    Sys --> Inv["【关键】Model.invoke / 提供商 API"]
    Inv --> Out["RunOutput / 流式 chunk"]
```

**【关键】节点说明：**

- **print_response / run**：用户可见的同步入口。
- **get_system_message**：系统提示拼装核心。
- **Model.invoke**：对模型提供商的实际请求。

## 关键源码文件索引

| 文件 | 作用 |
|------|------|
| `agno/agent/_messages.py` | `get_system_message()` L106+ |
| `agno/agent/agent.py` | `Agent` 运行与 CLI 输出 |
| `agno/models/` | 各厂商 `Model.invoke` |
