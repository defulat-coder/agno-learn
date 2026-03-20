# docker_tools.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Docker Tools
=============================

Demonstrates docker tools.
"""

import sys

from agno.agent import Agent

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------


# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    try:
        from agno.tools.docker import DockerTools

        # Example 1: Include specific Docker functions for container management
        container_tools = DockerTools(
            include_tools=[
                "list_containers",
                "start_container",
                "stop_container",
                "get_container_logs",
                "inspect_container",
            ]
        )

        # Example 2: Exclude dangerous functions (like remove operations)
        safe_docker_tools = DockerTools(
            exclude_tools=[
                "remove_container",
                "remove_image",
                "remove_volume",
                "remove_network",
            ]
        )

        # Example 3: Include all functions (default behavior)
        full_docker_tools = DockerTools()

        # Create agents with different tool configurations
        container_agent = Agent(
            name="Docker Container Agent",
            instructions=[
                "You are a Docker container management assistant.",
                "You can list, start, stop, and inspect containers.",
            ],
            tools=[container_tools],
            markdown=True,
        )

        safe_agent = Agent(
            name="Safe Docker Agent",
            instructions=[
                "You are a Docker management assistant with safe operations only.",
                "You can view and manage Docker resources but cannot delete them.",
            ],
            tools=[safe_docker_tools],
            markdown=True,
        )

        docker_agent = Agent(
            name="Full Docker Agent",
            instructions=[
                "You are a comprehensive Docker management assistant.",
                "You can manage containers, images, volumes, and networks.",
            ],
            tools=[full_docker_tools],
            markdown=True,
        )

        # Example 1: List running containers
        docker_agent.print_response("List all running Docker containers", stream=True)

        # Example 2: List all images
        docker_agent.print_response(
            "List all Docker images on this system", stream=True
        )

        # Example 3: Pull an image
        docker_agent.print_response("Pull the latest nginx image", stream=True)

        # Example 4: Run a container
        docker_agent.print_response(
            "Run an nginx container named 'web-server' on port 8080", stream=True
        )

        # Example 5: Get container logs
        docker_agent.print_response(
            "Get logs from the 'web-server' container", stream=True
        )

        # # Example 6: List volumes
        docker_agent.print_response("List all Docker volumes", stream=True)

        # Example 7: Create a network
        docker_agent.print_response(
            "Create a new Docker network called 'test-network'", stream=True
        )

        # Example 8: Stop and remove container
        docker_agent.print_response(
            "Stop and remove the 'web-server' container", stream=True
        )

        # Example 9: Inspect an image
        docker_agent.print_response("Inspect the nginx image", stream=True)

        # # Example 10: Build an image (uncomment and modify path as needed)
        # docker_agent.print_response(
        #     "Build a Docker image from the Dockerfile in ./app with tag 'myapp:latest'",
        #     stream=True
        # )

    except ValueError as e:
        print(f"\n[ERROR] Docker Tool Error: {e}")
        print("\nTroubleshooting steps:")

        if sys.platform == "darwin":  # macOS
            print("1. Ensure Docker Desktop is running")
            print("2. Check Docker Desktop settings")
            print("3. Try running 'docker ps' in terminal to verify access")

        elif sys.platform == "linux":
            print("1. Check if Docker service is running:")
            print("   systemctl status docker")
            print("2. Make sure your user has permissions to access Docker:")
            print("   sudo usermod -aG docker $USER")

        elif sys.platform == "win32":
            print("1. Ensure Docker Desktop is running")
            print("2. Check Docker Desktop settings")
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/91_tools/docker_tools.py`

## 概述

Docker Tools

本示例归类：**单 Agent**；模型相关类型：`（见源码 import）`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | 'Docker Container Agent' | `Agent(...)` |
| `markdown` | True | `Agent(...)` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ docker_tools.py      │  ──▶  │ Agent → get_run_messages → Model │
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
（主 `Agent(...)` 未传入可静态解析的 `description`/`instructions`/`system_message` 字符串；此时 system 由 `get_system_message()` 默认段与 `markdown` 等开关决定，请在 `agno/agent/_messages.py` 对照分段注释，或在运行中打印 `get_system_message` 返回值。）
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
