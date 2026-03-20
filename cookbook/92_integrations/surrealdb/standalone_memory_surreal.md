# standalone_memory_surreal.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
Standalone SurrealDB Memory Operations
"""

from agno.db.surrealdb import SurrealDb
from agno.memory import MemoryManager, UserMemory
from rich.pretty import pprint

# ---------------------------------------------------------------------------
# Setup
# ---------------------------------------------------------------------------
SURREALDB_URL = "ws://localhost:8000"
SURREALDB_USER = "root"
SURREALDB_PASSWORD = "root"
SURREALDB_NAMESPACE = "agno"
SURREALDB_DATABASE = "memories"

creds = {"username": SURREALDB_USER, "password": SURREALDB_PASSWORD}
db = SurrealDb(None, SURREALDB_URL, creds, SURREALDB_NAMESPACE, SURREALDB_DATABASE)


# ---------------------------------------------------------------------------
# Create Memory Manager
# ---------------------------------------------------------------------------
memory = MemoryManager(db=db)


# ---------------------------------------------------------------------------
# Run Example
# ---------------------------------------------------------------------------
def run_example() -> None:
    # Add a memory for the default user
    memory.add_user_memory(
        memory=UserMemory(memory="The user's name is John Doe", topics=["name"]),
    )
    print("Memories:")
    pprint(memory.get_user_memories())

    # Add memories for Jane Doe
    jane_doe_id = "jane_doe@example.com"
    print(f"\nUser: {jane_doe_id}")
    memory_id_1 = memory.add_user_memory(
        memory=UserMemory(memory="The user's name is Jane Doe", topics=["name"]),
        user_id=jane_doe_id,
    )
    memory_id_2 = memory.add_user_memory(
        memory=UserMemory(memory="She likes to play tennis", topics=["hobbies"]),
        user_id=jane_doe_id,
    )
    memories = memory.get_user_memories(user_id=jane_doe_id)
    print("Memories:")
    pprint(memories)

    # Delete a memory
    print("\nDeleting memory")
    assert memory_id_2 is not None
    memory.delete_user_memory(user_id=jane_doe_id, memory_id=memory_id_2)
    print("Memory deleted\n")
    memories = memory.get_user_memories(user_id=jane_doe_id)
    print("Memories:")
    pprint(memories)

    # Replace a memory
    print("\nReplacing memory")
    assert memory_id_1 is not None
    memory.replace_user_memory(
        memory_id=memory_id_1,
        memory=UserMemory(memory="The user's name is Jane Mary Doe", topics=["name"]),
        user_id=jane_doe_id,
    )
    print("Memory replaced")
    memories = memory.get_user_memories(user_id=jane_doe_id)
    print("Memories:")
    pprint(memories)


if __name__ == "__main__":
    run_example()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/92_integrations/surrealdb/standalone_memory_surreal.py`

## 概述

Standalone SurrealDB Memory Operations

本示例归类：**脚本/工具入口**；模型相关类型：`（见源码 import）`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| （见源码） | — | 请展开 `Agent` / `Team` 构造参数 |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ standalone_memory_surreal.py │  ──▶  │ Agent → get_run_messages → Model │
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
