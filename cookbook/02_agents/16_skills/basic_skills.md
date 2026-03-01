# basic_skills.py — 实现原理分析

> 源文件：`cookbook/02_agents/16_skills/basic_skills.py`

## 概述

本示例展示 Agno 的 **Skills（技能）**机制：通过 `Skills(loaders=[LocalSkills(dir)])` 从本地目录加载技能文件，将技能作为 Agent 的预置能力注入，使 Agent 能够执行超出标准工具范围的专业化任务（如代码审查）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Code Review Agent"` | Agent 名称 |
| `model` | `OpenAIResponses(gpt-5.2)` | Responses API |
| `skills` | `Skills(loaders=[LocalSkills(skills_dir)])` | 从本地目录加载技能 |
| `markdown` | `True` | Markdown 输出 |

## 核心模式：Skills 加载

```python
from agno.skills import LocalSkills, Skills
from pathlib import Path

# 技能目录：与当前文件同级的 sample_skills/ 目录
skills_dir = Path(__file__).parent / "sample_skills"

agent = Agent(
    name="Code Review Agent",
    model=OpenAIResponses(id="gpt-5.2"),
    skills=Skills(loaders=[LocalSkills(str(skills_dir))]),
    instructions=["You are a helpful assistant with access to specialized skills."],
    markdown=True,
)

agent.print_response(
    "Review this Python code and provide feedback:\n\n"
    "```python\n"
    "def calculate_total(items):\n"
    "    total = 0\n"
    "    for i in range(len(items)):\n"
    "        total = total + items[i]['price'] * items[i]['quantity']\n"
    "    return total\n"
    "```"
)
```

## Skills vs Tools 对比

| 特性 | Tools | Skills |
|------|-------|--------|
| 定义方式 | Python 函数/Toolkit 类 | 文件（通常为 YAML/JSON 格式的预置提示/逻辑） |
| 加载方式 | `tools=[...]` | `skills=Skills(loaders=[...])` |
| 来源 | 代码中定义 | 本地文件目录（`LocalSkills`） |
| 适用场景 | API 调用、数据处理 | 领域专业知识、复杂指令集 |

## Skills 加载流程

```
LocalSkills(skills_dir)
  → 扫描 sample_skills/ 目录
  → 加载技能文件（如 code_review.yaml 等）
  → 解析技能定义（名称、描述、提示等）
  → 注入到 Agent 的可用能力中
  → LLM 可以调用这些技能处理请求
```

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.2.4 | `add_name_to_context` | "Code Review Agent" | 是 |
| 3.3.2 | `instructions` | "You are a helpful assistant with access to specialized skills." | 是 |
| Skills | 来自 LocalSkills 文件 | 技能定义内容 | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/skills/__init__.py` | `Skills`, `LocalSkills` | Skills 加载器 |
| `agno/agent/agent.py` | `Agent(skills=...)` | Skills 注入入口 |
| `cookbook/02_agents/16_skills/sample_skills/` | 技能文件目录 | 实际技能定义 |
