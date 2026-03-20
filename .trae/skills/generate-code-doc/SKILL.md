---
name: generate-code-doc
description: Use when analyzing agno-cookbook Python source files to generate implementation principle documentation (.md). Use when the user asks to generate code analysis, implementation docs, or principle documentation for .py files.
---

# Agno 代码原理文档生成

分析 agno-cookbook 中的 Python 源文件，生成结构化的实现原理分析文档（.md），覆盖 Agent 配置、prompt 组装、API 请求、源码索引等。

## 适用场景

- 用户要求对某个 `.py` 文件生成原理分析文档
- 用户要求对某个目录批量生成所有 `.py` 文件的原理文档
- 用户提到"实现原理"、"代码分析"、"原理文档"等关键词

## 输出规范

- 文件名：`<源文件名>.md`（如 `few_shot_learning.py` → `few_shot_learning.md`）
- 输出位置：与源 `.py` 文件同目录
- 语言：中文

## 文档模板结构

每个 `.md` 文件必须包含以下章节，按顺序排列：

### 1. 标题与源文件引用

```markdown
# <文件名>.py — 实现原理分析

> 源文件：`<相对路径>/<文件名>.py`
```

### 2. 概述

一段话总结该示例展示的 Agno 核心机制，加粗关键特性名。末尾附「核心配置一览」表格：

```markdown
## 概述

本示例展示 Agno 的 **`<核心特性名>`** 机制：<一句话描述原理和用途>。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"xxx"` | Agent 名称 |
| `model` | `OpenAIChat(id="gpt-4o")` | Chat Completions API |
| `instructions` | ... | ... |
| `tools` | ... | ... |
| ... | ... | ... |
```

配置表规则：
- 列出 Agent 构造函数的**所有显式参数**
- 对未设置的重要参数也要列出（值填 `None`，说明填"未设置"）
- model 列标注 API 类型（Chat Completions / Responses）

### 3. 架构分层

ASCII 文本图，展示用户代码层 → agno.agent 层 → 模型层的数据流：

```markdown
## 架构分层

​```
用户代码层                agno.agent 层
┌──────────────────┐    ┌──────────────────────────────────┐
│ <文件名>.py      │    │ Agent._run()                     │
│                  │    │  ├ _messages.py                  │
│ <关键参数>       │───>│  │  get_system_message()          │
│                  │    │  │    → <处理逻辑概要>            │
│                  │    │  │                                │
│                  │    │  │  get_run_messages()            │
│                  │    │  │    <消息组装概要>              │
└──────────────────┘    └──────────────────────────────────┘
                                │
                                ▼
                        ┌──────────────┐
                        │ <Model类>    │
                        │ <model_id>   │
                        └──────────────┘
​```
```

规则：
- 左侧为用户代码层，展示关键参数
- 右侧为 agno 内部处理层，展示调用链
- 底部为模型层
- 如涉及工具、知识库等额外组件，在右侧或底部扩展

### 4. 核心组件解析

逐个分析该文件使用的 Agno 核心机制，附源码引用：

```markdown
## 核心组件解析

### <组件名1>

`<组件名>` 在 `<函数名>()`（`<文件>:<行号>`）中处理：

​```python
# 关键源码片段（带中文注释）
​```

<解释这段代码的作用和原理>

### <组件名2>
...
```

规则：
- 每个组件单独一个 `###` 子章节
- 引用 agno 源码时标注文件路径和行号
- 代码片段保留关键逻辑，添加中文注释
- 如有对比场景（如有/无某配置），用表格对比

#### 深度原理（必写，避免「点到为止」）

在「核心组件解析」中或紧随其后，用一小节 **`### 运行机制与因果链`**（或等价标题）写清下面四类问题（与示例相关则写，无关则一句说明不适用）：

1. **数据从哪进、到哪出**：用户/API/工具/会话状态在本示例中的一条主路径；指出关键入口函数（如 `Agent.run` → `run_agent` → `get_run_messages`）。
2. **状态与副作用**：本示例是否写入 session、db、knowledge、全局单例（如 `set_cancellation_manager`）；若中断/重试，哪些会重复执行。
3. **关键分支**：列出 1～3 个「若配置为 A 则走分支 X，若为 B 则走 Y」的对照（可用小表）。
4. **与相邻示例的差异**：一句话说明本文件在 cookbook 主题下的定位（例如「在 `basic_agent` 之上只多了 xxx」）。

篇幅目标：让读者**不看源码也能回答**「这个示例到底演示了什么机制、该机制在框架里卡在哪一层」。

### 5. System Prompt 组装（须把「最终提示词」讲清楚）

本节目标是让读者**理解模型在跑起来之前到底收到了什么系统侧约束**，以及**每一段文字从哪来的、起什么作用**。禁止用「（含 description；无 instructions）」这类**括号摘要**代替真实可读的提示词正文。

表格列出 system prompt 的所有组成部分及其在本文件中的状态：

```markdown
## System Prompt 组装

| 序号 | 组成部分 | 本文件中的值/来源 | 是否生效 |
|------|---------|-----------------|---------|
| 1 | `description` | ... | 是/否 |
| 2 | `role` | ... | 是/否 |
| 3 | `instructions` | ... | 是/否 |
| 4.1 | `markdown` | ... | 是/否 |
| 4.2 | `add_datetime_to_context` | ... | 是/否 |
| 4.3 | `add_location_to_context` | ... | 是/否 |
| 4.4 | `add_name_to_context` | ... | 是/否 |
| 5-12 | 其余段落 | ... | 否 |

### 拼装顺序与源码锚点

用有序列表写出**默认拼装路径**下各段进入 `system_message_content` 的**先后顺序**，并标注 `get_system_message()`（`agno/agent/_messages.py`）中对应片段的**注释编号**（如 `# 3.1`、`# 3.3.3`）或函数分支（如 `if agent.system_message is not None` 早退）。若本示例走了**非默认路径**（例如直接传入 `system_message`、或 `build_context=False`），必须写明从哪一步起与默认不同。

### 还原后的完整 System 文本

​```text
<此处写入「还原后的」完整 system 文本：应逐字来自示例中可确定的字符串，或按 _messages.py 逻辑对默认拼装结果做忠实重组>

若示例里有多条候选用户输入，任选文档中**明确写出**的一条用户消息作为「当前 run」的参照，并在此说明选取的是哪一句。
​```

**还原规则（必遵守）：**

1. **能静态还原就必须还原**：凡在 cookbook `.py` 里以字面量给出的 `instructions`、`description`、`system_message`、expected_output 文案等，必须**原样**出现在上面的 `text` 代码块中（保持换行与标点）。
2. **无法静态还原时要写清原因**，并给出**如何验证**（任选其一写清楚即可）：
   - 在文档中说明：于 `get_system_message()` 返回前打断点 / 临时 `print(message.content)`；或
   - 说明依赖运行时 session/knowledge/tools 解析结果，本示例未固定，故只列**结构**不伪造正文。
3. **绝对禁止**：用「（含 xxx；无 yyy）」替代整块正文；若确实无 system message（例如返回 `None`），应用单独小节说明**为何为 None**（引用分支条件），而不是伪造一段占位括号文。

### 段落释义（模型视角）

用 3～8 条 bullet，说明**每一段 system 文本对模型行为的约束**（例如：角色边界、输出格式、工具使用策略、何时必须引用知识库等）。这里要用**教学口吻**讲透「为什么要这样写」，而不是重复表格列名。

### 与 User / Developer 消息的边界

简要说明：本 run 中 system 与用户消息如何分工；若模型适配器使用 `developer` 而非 `system`（视具体 `Model` 与消息转换逻辑而定），在下一节「完整 API 请求」中保持一致，并在此交叉引用。
```

**针对非 Agent 示例**：若源码为纯脚本、Team、Workflow、仅工具函数而无 `Agent(...)`，本节改为说明「不存在单一 Agent 的 `get_system_message`」的原因，并说明提示词/指令出现在何处（例如 Team 成员 Agent、工作流步骤字符串）；不得硬套空表格。

### 6. 完整 API 请求

模拟**实际**使用的模型适配器向提供商发起的调用（**不要抄固定模板**）：

```markdown
## 完整 API 请求

​```python
# 先根据 cookbook 中的 Model 类判定：
# - OpenAI Chat Completions 系 → chat.completions.create + messages[]
# - OpenAI Responses 系 → responses.create + input=[...] 等
# - 其他厂商按 agno/models/<厂商>/ 中 invoke 实现为准

<与当前示例一致的调用形态，含 model id、stream、tools 等>
​```

> <补充说明：system/developer 角色如何映射、历史消息从哪来、本示例是否流式>
```

规则：
- **先 Read** 示例使用的 `Model` 子类及其 `invoke`/`ainvoke`（或 `get_client`）实现，再决定用 `messages` 还是 `input`、role 名是 `system` 还是 `developer`。
- 展示**与一次典型 run 等价**的请求结构；若示例主要演示非 LLM 路径（如仅取消运行），说明 LLM 调用为附带或给出最小示例即可。
- 用注释标注每部分消息的来源（`get_system_message`、`history`、`user` 输入等）。
- 末尾用引用块说明与第 5 节「还原后的 System 文本」的对应关系。

### 7. Mermaid 流程图

**不要使用** Mermaid 的 `style` / `classDef` / `fill` 等**样式指令**（避免渲染差异、深色主题下不可读、部分平台不支持）。**关键节点**改在**节点文案里标注**，读者一眼能识别。

```markdown
## Mermaid 流程图

​```mermaid
flowchart TD
    A["用户代码<br/>agent.print_response(...)"] --> B["Agent._run()"]
    B --> C["【关键】get_system_message()"]

    subgraph SystemPrompt["System Prompt 组装"]
        C --> C1[...]
        ...
    end

    ... --> E["【关键】<Model>.invoke()"]
    E --> F["模型响应"]
    F --> G["流式输出"]
​```
```

**关键节点标注规则：**

- 在**本示例真正演示的机制**对应节点上，用前缀 **`【关键】`**（或同一文档内统一用 **`[K]`**，二选一）写在节点标签**第一行或单独一行**（可与 `<br/>` 换行连用），例如：`["【关键】cancel_run<br/>写入取消状态"]`。
- 全文 **1～4 个**关键节点即可，避免处处标「关键」；入口、出口不必强行标注，除非本示例重点就是入口/出口行为。
- 若关键步骤是一段子图，可用 `subgraph` 标题标出，例如 `subgraph K1["【关键】工具调用循环"]`。
- 图后可选加**简短说明**（无序列表）：每个「【关键】」节点**对应什么机制、为何关键**（一句即可），避免只贴图不讲理。

**结构规则：**

- 用 `subgraph` 分组相关步骤；`subgraph` 的 id 使用英文/下划线（**不含空格**），中文可放在引号标题里。
- 如涉及条件分支，用菱形节点 `{{判断条件}}`。
- 如涉及工具调用循环，在图中展示 agentic loop。
- 节点 ID 仍遵循**不含空格**（驼峰或下划线）。

### 8. 关键源码文件索引

```markdown
## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `<属性名>` L<行号> | <作用> |
| `agno/agent/_messages.py` | `<函数名>()` L<行号> | <作用> |
| ... | ... | ... |
```

规则：
- 只列与本文件核心机制直接相关的源码
- 标注行号（格式 `L<起始>-<结束>` 或 `L<行号>`）
- 按调用链顺序排列

## 分析工作流程

### 单文件模式

1. **Read** 目标 `.py` 文件，理解代码逻辑
2. **定位核心 Agno 特性**：识别 Agent 构造参数中的关键机制
3. **追踪 agno 源码**：
   - `Grep` 搜索 agno 库中相关属性/函数的实现
   - `Read` 关键源码文件（`_messages.py`、`agent.py`、`_tools.py` 等）
   - 记录行号
4. **组装文档**：按模板结构依次生成各章节
5. **Write** 输出到同目录的 `.md` 文件

### 批量模式（整个目录）

#### 扫描范围与排除规则（失败率最高的环节）

对 **`cookbook/` 整树**做 `rglob("*.py")` 时，若不排除虚拟环境，会把 **`cookbook/.venv/.../site-packages`** 下已安装的第三方包与 **`agno` 安装副本** 扫进来，导致：

- 路径数量虚高（例如 **三千级** vs 排除后 **一千多级** 真实示例）；
- 并行分片被 **venv 路径** 占满，子代理大量时间在「跳过非示例」；
- 若误在 `site-packages` 旁生成 `.md`，会**污染依赖目录**（严禁）。

**必须在枚举阶段排除**（路径任一段匹配即跳过）：

- `.venv`、`venv`、`.venvs`
- `site-packages`
- `__pycache__`
- （按需）`node_modules`、`dist`、`build`

**结论**：批量任务的第一步永远是 **「只保留真正的 cookbook 示例 `.py`」**，再谈分片与并行。

#### 推荐流程（可续跑、可并行）

1. **枚举**：在已排除 venv 的前提下收集所有目标 `.py`（仍排除 `__init__.py` 除非用户明确要求）。
2. **（推荐）仅缺文档模式**：对每条路径若已存在同目录 `<basename>.md` 且用户未要求覆盖，则**不加入待处理列表**。得到「待生成」列表便于**断点续跑**与第二轮补洞。
3. **统计数量**，按默认或用户指定并行度分片（见下表）。
4. **Manifest 落盘**：当路径多于 **~50 条** 时，**不要**把几千行路径塞进单条提示词。应写入一个或多个文本文件（每行一个绝对路径或仓库相对路径），例如项目内 **`.context/generate-code-doc-*/manifest_partK.txt`**（`.context` 常已 gitignore）。每个 subagent **只读取自己被分配的那一个 manifest**。
5. **分片策略**：路径已按 **稳定排序**（如字典序）后，可用 **按序号轮询**（`i % N`）或 **连续切块** 分到 N 个子任务，使各份**工作量接近**。
6. **执行**：每个子代理顺序处理 manifest 中每一行；**禁止**把 `site-packages` 里的 `.py` 当作 cookbook 示例生成文档。

| Python 文件数 | 策略 | Agent 数（默认） |
|--------------|------|---------|
| ≤ 5 | 直接处理，不启动 Agent | 0 |
| 6-15 | 启动 2 个 Agent 并行 | 2 |
| 16-30 | 启动 3 个 Agent 并行 | 3 |
| > 30 | 启动 4 个 Agent 并行 | 4 |

用户若明确要求更多并行子任务（例如 10 个 subagent），在**不超过执行环境并发上限**的前提下，可将 **「待生成列表」** 均匀切分为 N 份；**并行度不降低质量要求**（行号真实、System 还原规则仍有效）。

**第二轮补全**：若第一轮因清单含 venv、超时或中断导致部分 `.py` 仍无 `.md`，应 **重新扫描「排除 venv + 缺 `.md`」** 生成新 manifest，再分片执行，**不要**在含 venv 的旧清单上续跑。

#### 自动化批量与质量层级

超大批量下用脚本辅助生成时，易出现：**「核心配置一览」填占位符**、`instructions` 为**变量名**时无法静态还原长文案。技能要求：

- **要么**脚本产出的文档在文首或文末用一行标注 **`<!-- 批量生成骨架：配置表与 System 还原需对照 .py 精修 -->`**（或等价说明），便于后续人工或 LLM 精修；
- **要么**脚本必须尽量用 **AST 抽取** `Agent(...)` 中的**字面量**参数填入表格与「还原后的 System 文本」，避免空表。

子代理**不得**在不知情的情况下把「占位表 + 虚构段落」冒充为已按技能完成的终稿。

#### 分组与提示词模板

1. 按文件数均匀分配到各 Agent（或按 manifest 已分片结果一对一）
2. 每个 Agent 的提示词模板：

```
（若路径很多：改为「读取 manifest 文件 <路径>，每行一个 .py，共 <N> 行」，不要内联数千行路径。）

分析以下 agno-cookbook Python 文件，为每个文件生成实现原理文档。

文件列表（共 <N> 个）：
- <文件路径1>
- <文件路径2>
- ...

对每个文件执行以下步骤：
1. Read 文件，理解代码逻辑
2. 识别使用的 Agno 核心特性（Agent 参数、tools、knowledge、storage 等）
3. Grep agno 源码追踪相关实现（_messages.py、agent.py、_tools.py 等），记录行号
4. 按以下模板结构生成 .md 文件（Write 到源文件同目录）：
   - 标题 + 源文件引用
   - 概述 + 核心配置一览表格
   - 架构分层（ASCII 图）
   - 核心组件解析（附源码片段和行号）+ **运行机制与因果链**
   - System Prompt：表格 + **拼装顺序与源码锚点** + **还原后的完整 System 文本**（禁止括号占位）+ **段落释义** + 与 user 消息边界
   - 完整 API 请求（**按实际 Model 适配器**书写，勿套固定 chat.completions 模板）
   - Mermaid 流程图（**无** `style`/填色；关键节点用 **`【关键】`** 标在节点文案内）
   - 关键源码文件索引

语言：中文
输出位置：与源 .py 文件同目录，文件名为 <源文件名>.md

【重要】处理完列表中所有文件后再报告，不要中途停止。
完成后报告已生成的文件数和文件列表。
```

## Agno 源码追踪指南

分析时需要追踪的 agno 核心源码文件：

| 机制 | 关键源码文件 | 关键函数 |
|------|------------|---------|
| System Prompt | `agno/agent/_messages.py` | `get_system_message()` |
| 消息组装 | `agno/agent/_messages.py` | `get_run_messages()` |
| Agent 属性 | `agno/agent/agent.py` | Agent 类定义 |
| 工具处理 | `agno/agent/_tools.py` | `get_tools()` |
| 可调用工厂 | `agno/utils/callables.py` | `invoke_callable_factory()` |
| 知识库 | `agno/knowledge/` | 各 KnowledgeBase 子类 |
| 存储 | `agno/storage/` | 各 Storage 子类 |
| 记忆 | `agno/memory/` | Memory 类 |
| 模型适配 | `agno/models/` | 各模型的 `invoke()` / `ainvoke()` |
| Team | `agno/team/` | Team 类 |
| Workflow | `agno/workflow/` | Workflow 类 |

追踪技巧：
- 用 `Grep` 搜索属性名在 `_messages.py` 中的处理位置
- 关注 `get_system_message()` 的分段注释（`# 3.1`, `# 3.2` 等）
- 记录行号时使用 `L<行号>` 格式

## 注意事项

- **枚举范围**：对 `cookbook/` 全树批量时**必须排除** `.venv` / `site-packages`，否则路径数量与分片会失真，且不得在 venv 内写 `.md`
- **不要虚构行号**：必须通过实际 Read/Grep 确认源码行号
- **不要遗漏配置项**：核心配置一览表要列出所有显式参数（批量骨架若暂缺，须按「自动化批量与质量层级」标注待精修）
- **API 请求要准确**：先识别示例中的 `Model` 子类，再写对应的 `invoke` 形态；不要默认 `chat.completions.create`
- **最终提示词必须可教**：第 5 节中「还原后的完整 System 文本」须为真实文本或明确说明无法静态还原的原因与验证方式；**禁止**用括号摘要糊弄
- **讲透原理**：必须包含「运行机制与因果链」，避免只列参数不讲因果
- **Mermaid**：**禁止** `style` / `classDef` / 填色；关键节点用节点文案前缀 **`【关键】`**（或统一的 `[K]`）标出，并在图后可选附一句说明
- **Mermaid 节点 ID 不能有空格**：用驼峰或下划线命名
- **跳过已存在的 .md 文件**：批量模式下，如果同名 .md 已存在则跳过

## 批量生成后的经验复盘（可选）

若用户关心技能迭代，执行者可简要记录：

- **行号漂移**：`libs/agno/agno` 更新后旧文档行号是否仍可用
- **高频错误**：API 节误用 `chat.completions`；System 节用括号摘要；**标题误写成 `.md` 而非 `.py`**
- **全库任务**：是否误扫 venv；第二轮是否用「仅缺 `.md`」清单补洞
- **模板维护**：`_messages.py` 内分段注释编号是否变更，需在「拼装顺序」小节同步