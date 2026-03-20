# 13_pdf_input.py — 实现原理分析

<!-- cookbook-py-source:start -->
## 完整源码

```python
"""
PDF Understanding - Read and Analyze Documents
================================================
Pass PDF documents to Gemini for reading and analysis. No parsing libraries needed.

Key concepts:
- File(url=..., mime_type="application/pdf"): Pass a PDF from a URL
- File(filepath=..., mime_type="application/pdf"): Pass a local PDF
- Native capability: No PyPDF, pdfplumber, or other parsing libraries needed
- Layout-aware: The model understands tables, columns, and formatting

Example prompts to try:
- "Summarize the contents of this document"
- "What are the main recipes in this cookbook?"
- "Extract all the key findings from this research paper"
"""

from agno.agent import Agent
from agno.media import File
from agno.models.google import Gemini

# ---------------------------------------------------------------------------
# Agent Instructions
# ---------------------------------------------------------------------------
instructions = """\
You are a document analysis expert. Read documents thoroughly
and provide clear summaries.

## Rules

- Summarize the main points first
- Note any tables or structured data
- Highlight actionable information\
"""

# ---------------------------------------------------------------------------
# Create Agent
# ---------------------------------------------------------------------------
doc_reader = Agent(
    name="Document Reader",
    model=Gemini(id="gemini-3-flash-preview"),
    instructions=instructions,
    markdown=True,
)

# ---------------------------------------------------------------------------
# Run Agent
# ---------------------------------------------------------------------------
if __name__ == "__main__":
    doc_reader.print_response(
        "Summarize the contents of this document and suggest a recipe from it.",
        files=[
            File(
                url="https://agno-public.s3.amazonaws.com/recipes/ThaiRecipes.pdf",
                mime_type="application/pdf",
            )
        ],
        stream=True,
    )

# ---------------------------------------------------------------------------
# More Examples
# ---------------------------------------------------------------------------
"""
PDF input methods:

1. From URL
   files=[File(url="https://example.com/report.pdf", mime_type="application/pdf")]

2. From local file
   files=[File(filepath="path/to/report.pdf", mime_type="application/pdf")]

3. Multiple PDFs
   files=[
       File(url="...", mime_type="application/pdf"),
       File(filepath="...", mime_type="application/pdf"),
   ]

4. With structured output (extract data from PDFs)
   class Report(BaseModel):
       title: str
       key_findings: List[str]
       recommendations: List[str]

   agent = Agent(model=Gemini(...), output_schema=Report)
   result = agent.run("Extract findings", files=[...])

Use cases for music/film/gaming:
- Parse music licensing contracts
- Extract requirements from game design documents
- Analyze film scripts for scene breakdowns
"""
```

<!-- cookbook-py-source:end -->

> 源文件：`cookbook/gemini_3/13_pdf_input.py`

## 概述

PDF Understanding - Read and Analyze Documents

本示例归类：**单 Agent**；模型相关类型：`Gemini`。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | 'Document Reader' | `Agent(...)` |
| `model` | Gemini(id='gemini-3-flash-preview'…) | `Agent(...)` |
| `instructions` | 'You are a document analysis expert. Read documents thoroughly\nand provide clear summaries.\n\n## Rules\n\n- Summarize the...' | `Agent(...)` |
| `markdown` | True | `Agent(...)` |
| （Model 类） | `Gemini` | `agno.models` |

## 架构分层

```
用户 / cookbook 示例              Agno 框架
┌──────────────────────┐         ┌────────────────────────────────┐
│ 13_pdf_input.py      │  ──▶  │ Agent → get_run_messages → Model │
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
--- instructions ---
You are a document analysis expert. Read documents thoroughly
and provide clear summaries.

## Rules

- Summarize the main points first
- Note any tables or structured data
- Highlight actionable information
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
