---
name: translate-to-chinese
description: Use when translating Python project files (comments, docstrings, instructions, markdown) from English to Chinese. Use when the user asks to translate code comments, README, documentation, or AI prompt instructions to Chinese.
---

# 项目文件中英翻译

将 Python 项目中的英文内容翻译为中文，覆盖注释、文档字符串、instructions prompt、markdown 文件等。

## 翻译范围

### Python 文件（.py）

需要翻译的内容：
- 模块顶部的 docstring（`"""..."""`）
- 行内注释（`# ...`）
- 函数/类的 docstring
- `instructions = """..."""` 变量中的 AI prompt 内容
- `additional_instructions` / `compression_prompt` / `message` 等 prompt 类变量
- 打印输出文本（`print()` 中的字符串）
- 用户查询示例（示例对话内容）
- 文件底部 `"""..."""` 中的说明文字

不翻译的内容：
- 代码逻辑、变量名、函数名、类名
- import 语句
- 字符串常量（非注释/文档用途的，如 API key、URL）
- Pydantic Field 的 `description` 参数（保留英文，因为是给模型的 schema）
- `role` 参数值（保留英文）
- 文件路径和命令（如 `python xxx.py`）
- 代码块中的代码（仅翻译注释）

### Markdown 文件（.md）

全文翻译，包括：
- 标题、段落、列表
- 表格内容
- 代码块中的注释（代码本身不翻译）

## 工作流程

### 第一步：分析文件分布（所有规模通用）

1. 并行 `Glob` 查找 `.py` 和 `.md` 文件
2. 统计 Python 文件数并按子目录分组
3. 根据 Python 文件总数选择下面的策略

### 小型目录（< 20 个 Python 文件）
**直接处理，不启动 Agent**
1. Python 文件：`Read` + 多次 `Edit`，逐段翻译
2. Markdown 文件：`Read` + `Write` 整体写入

### 中型目录（20-50 个 Python 文件）
**启动 2-3 个 Agent 并行**
1. 将 Python 文件按子目录分成 2 组，各启动 1 个 Python Agent
2. 启动 1 个 Markdown Agent
3. 所有 Agent 并行运行

### 大型目录（50-100 个 Python 文件）
**启动 3-4 个 Agent 并行**
1. 将 Python 文件按子目录分成 3 组，各启动 1 个 Python Agent
2. 启动 1 个 Markdown Agent
3. 所有 Agent 并行运行

### 超大型目录（> 100 个 Python 文件）
**启动 4-5 个 Agent 并行**
1. 将 Python 文件按子目录分成 4 组，各启动 1 个 Python Agent
2. 启动 1 个 Markdown Agent
3. 所有 Agent 并行运行
4. 如果个别 Agent 未完成，使用 `resume` 继续推进

### Python 文件分组策略（关键）

**按子目录均匀分配**：
1. 列出所有子目录及其 Python 文件数
2. 按文件数从大到小排序
3. 使用贪心算法将子目录分配到 N 个组，使每组文件数尽量均衡
4. 根目录的文件作为独立组或并入最小的组
5. **每个 Agent 给出明确的文件列表**，避免重复翻译

**示例**（假设 80 个 Python 文件，分 3 组）：
```
子目录分布：
  root/      (8 个)
  agents/    (25 个)
  tools/     (22 个)
  models/    (15 个)
  utils/     (10 个)

分组：
  Agent 1: agents/（25 个）
  Agent 2: tools/（22 个）
  Agent 3: root/ + models/ + utils/（33 个）
```

### Agent 提示词模板

**Python 文件 Agent（每个分组一个 Agent）：**
```
翻译以下 Python 文件的英文内容为中文。

文件列表（共 <N> 个）：
- <文件路径1>
- <文件路径2>
- ...

需要翻译：
- 模块顶部 docstring
- 行内注释 # ...
- 函数/类 docstring
- instructions/additional_instructions 等 prompt 变量内容
- print() 中的输出文本
- 用户查询示例

不翻译：
- 代码逻辑、变量名、import
- Pydantic Field description
- role 参数值

技术术语统一：Agent、Team（团队）、Workflow（工作流）、Knowledge（知识库）、Storage（存储）、Memory（记忆）、Tool（工具）

使用 Edit 逐段替换。

【重要】继续翻译直到列表中所有文件完成，不要中途停止总结。即使文件很多也要全部处理完。
完成后报告已翻译的文件数和文件列表。
```

**Markdown 文件 Agent：**
```
翻译以下 Markdown 文件的英文内容为中文。

文件列表（共 <N> 个）：
- <文件路径1>
- <文件路径2>
- ...

全文翻译：标题、段落、列表、表格。代码块中的注释也翻译，代码本身不翻译。

技术术语保留英文或附中文说明：Agent、Team（团队）、Workflow（工作流）等。

使用 Write 直接写入翻译后内容。完成后报告翻译的文件数。
```

**关键实践**：
- **每个 Python Agent 给出明确文件列表**，避免重复翻译
- 多个 Python Agent + 1 个 Markdown Agent 全部并行启动
- Markdown Agent 通常能一次性完成所有文件（Write 效率高）
- 如果某个 Python Agent 未完成全部文件，使用 `resume` 继续推进

## 翻译原则

- 技术术语保留英文或附中文括号说明，如 Agent、Workflow、Knowledge Base（知识库）
- 翻译要自然通顺，不生硬直译
- 保留原有代码结构、缩进、格式不变
- **用户可见文本优先翻译**：示例提示、print 输出、错误消息
- 常用术语统一：
  - Agent -> Agent
  - Team -> 团队
  - Workflow -> 工作流
  - Knowledge -> 知识库
  - Storage -> 存储
  - Memory -> 记忆
  - Guardrail -> 护栏
  - Tool -> 工具
  - Instructions -> 指令
  - Embedder -> 嵌入器
  - Vector DB -> 向量数据库
  - Hybrid search -> 混合搜索
  - Structured output -> 结构化输出
  - Human in the loop -> 人机协作
  - Session -> Session
  - State -> State
  - RAG -> RAG
  - Multimodal -> 多模态
  - Parser -> Parser
  - Hook -> Hook
  - Event -> 事件
  - Run -> Run
  - Step -> 步骤

## 性能优化与实战经验

### 核心优化：多 Agent 并行处理 Python 文件

**旧方案（慢）**：1 个 Python Agent 处理所有文件，需要 2-5 轮 resume
**新方案（快）**：按子目录拆分为多个 Python Agent，全部并行启动

**效果对比**：

| 目录 | Python 文件数 | 旧方案（1 Agent） | 新方案（多 Agent 并行） |
|------|-------------|------------------|----------------------|
| 03_teams | 118 | 2 轮 resume | 4 Agent 并行，无需 resume |
| 04_workflows | 79 | 3 轮 resume | 3 Agent 并行，无需 resume |
| 05_agent_os | 168 | 5 轮 resume | 4 Agent 并行，偶尔 1 轮 resume |
| 07_knowledge | 137 | 3 轮 resume | 4 Agent 并行，无需 resume |

### Agent 数量速查表

| Python 文件数 | Python Agent 数 | Markdown Agent 数 | 总 Agent 数 |
|--------------|----------------|-------------------|------------|
| < 20 | 0（直接处理） | 0（直接处理） | 0 |
| 20-50 | 2 | 1 | 3 |
| 50-100 | 3 | 1 | 4 |
| 100+ | 4 | 1 | 5 |

### 分组注意事项
- **每个 Agent 明确给出文件列表**：避免多个 Agent 翻译同一文件
- **按子目录分组而非逐文件分配**：同一子目录的文件放在同一个 Agent，减少上下文切换
- **文件数尽量均衡**：使用贪心算法分配，避免某个 Agent 负载过重
- **根目录文件并入最小组**：根目录通常文件最少

### Resume 策略（仅超大型目录偶尔需要）

当某个 Python Agent 未完成全部文件时：
1. 查看该 Agent 返回的已完成列表
2. 用 `resume` 参数继续推进，提示"继续翻译剩余文件"
3. 通常只需 1 轮 resume

**关键发现**：
- 多 Agent 并行后，每个 Agent 分到的文件数大幅减少（25-40 个），基本一次完成
- Markdown Agent 无需拆分，50+ 文件通常一次完成（Write 效率高）
- 增量翻译天然支持：Agent 会自动检测已翻译文件并跳过

### 实战经验总结

| 目录 | Python 文件数 | Markdown 文件数 | Python 轮数 | Markdown 轮数 |
|------|--------------|----------------|------------|--------------|
| 00_quickstart | 13 | 3 | 直接处理 | 直接处理 |
| 11_memory | 16 | 7 | 1 Agent | 1 Agent |
| 10_reasoning | 63 | 31 | 1 Agent | 1 Agent |
| 03_teams | 118 | 48 | 2 Agent（旧）/ 4 Agent（新） | 1 Agent |
| 04_workflows | 79 | 52 | 3 Agent（旧）/ 3 Agent（新） | 1 Agent |
| 05_agent_os | 168 | 66 | 5 Agent（旧）/ 4 Agent（新） | 1 Agent |
| 07_knowledge | 137 | 63 | 3 Agent（旧）/ 4 Agent（新） | 1 Agent |

## 常见问题

**Q: 多个 Python Agent 会不会翻译同一个文件？**
A: 不会。每个 Agent 有明确的文件列表，互不重叠。

**Q: Markdown Agent 为什么不需要拆分？**
A: 因为使用 Write 工具整体写入，效率极高，50+ 文件也能一次完成。

**Q: 某个目录之前已部分翻译，怎么处理？**
A: 直接按正常流程启动即可。Agent 会自动检测已翻译文件并跳过。

**Q: 如何判断分几个 Agent？**
A: 看 Python 文件总数：20-50 用 2 个，50-100 用 3 个，100+ 用 4 个。
