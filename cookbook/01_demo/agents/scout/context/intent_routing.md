# intent_routing.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""Load intent routing rules for the system prompt."""

import json
from pathlib import Path
from typing import Any

from agno.utils.log import logger

from ..paths import ROUTING_DIR


def load_intent_rules(routing_dir: Path | None = None) -> dict[str, Any]:
    """Load intent routing rules from JSON files."""
    if routing_dir is None:
        routing_dir = ROUTING_DIR

    rules: dict[str, Any] = {
        "intent_mappings": [],
        "source_preferences": [],
        "common_gotchas": [],
    }

    if not routing_dir.exists():
        return rules

    for filepath in sorted(routing_dir.glob("*.json")):
        try:
            with open(filepath) as f:
                data = json.load(f)
            for key in rules:
                if key in data:
                    rules[key].extend(data[key])
        except (json.JSONDecodeError, OSError) as e:
            logger.error(f"Failed to load {filepath}: {e}")

    return rules


def build_intent_routing(routing_dir: Path | None = None) -> str:
    """Build intent routing context string for system prompt."""
    rules = load_intent_rules(routing_dir)
    lines: list[str] = []

    # Intent mappings
    if rules["intent_mappings"]:
        lines.append("## INTENT ROUTING\n")
        lines.append("When the user asks about:")
        lines.append("")
        for mapping in rules["intent_mappings"]:
            intent = mapping.get("intent", "Unknown")
            primary = mapping.get("primary_source", "unknown")
            fallbacks = mapping.get("fallback_sources", [])
            reasoning = mapping.get("reasoning", "")

            lines.append(f"**{intent}**")
            lines.append(f"  - Primary: `{primary}`")
            if fallbacks:
                lines.append(f"  - Fallback: {', '.join(f'`{f}`' for f in fallbacks)}")
            if reasoning:
                lines.append(f"  - Why: {reasoning}")
            lines.append("")

    # Source preferences
    if rules["source_preferences"]:
        lines.append("## SOURCE STRENGTHS\n")
        for pref in rules["source_preferences"]:
            source = pref.get("source", "Unknown")
            best_for = pref.get("best_for", [])
            search_first = pref.get("search_first_when", [])

            lines.append(f"**{source}**")
            if best_for:
                lines.append(f"  - Best for: {', '.join(best_for)}")
            if search_first:
                lines.append(f"  - Search first when: {'; '.join(search_first)}")
            lines.append("")

    # Common gotchas
    if rules["common_gotchas"]:
        lines.append("## COMMON GOTCHAS\n")
        for gotcha in rules["common_gotchas"]:
            issue = gotcha.get("issue", "Unknown")
            description = gotcha.get("description", "")
            solution = gotcha.get("solution", "")

            lines.append(f"**{issue}**")
            if description:
                lines.append(f"  - {description}")
            if solution:
                lines.append(f"  - Solution: {solution}")
            lines.append("")

    return "\n".join(lines)


INTENT_ROUTING = load_intent_rules()
INTENT_ROUTING_CONTEXT = build_intent_routing()
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/01_demo/agents/scout/context/intent_routing.py`

## 概述

从 **`knowledge/routing/*.json`** 加载 **intent 映射、源偏好、常见坑**，**`build_intent_routing`** 格式化为字符串 **`INTENT_ROUTING_CONTEXT`**，嵌入 **`scout/agent.py`** 的 **`INSTRUCTIONS` f-string**（`---` 后 `{INTENT_ROUTING_CONTEXT}`）。

**核心配置一览：** 无 Agent。

## 架构分层

```
ROUTING_DIR/*.json → load_intent_rules → build_intent_routing → Scout instructions
```

## 核心组件解析

见 `build_intent_routing`（`intent_routing.py` L39+）：将结构化规则渲染为 Markdown 供模型遵循。

### 运行机制与因果链

静态 import 时拼入 instructions；改 JSON 需重启进程生效。

## System Prompt 组装

作为 **instructions 子串**，影响 **意图→路径** 的 prior。

### 还原后的完整 System 文本

取决于 `knowledge/routing` 下 JSON；空目录时规则对象各列表为空（`L17-L21`）。

## 完整 API 请求

无。

## Mermaid 流程图

```mermaid
flowchart LR
    A["routing/*.json"] --> B["【关键】build_intent_routing"]
    B --> C["INTENT_ROUTING_CONTEXT"]
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `intent_routing.py` | `build_intent_routing` L39 | 路由上下文 |
