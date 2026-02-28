---
name: migrate-to-glm
description: Use when migrating Agno project code from Gemini, OpenAIResponses, or OpenAIChat to GLM model (OpenAILike). Use when switching models, replacing Gemini/OpenAI with GLM, or configuring OpenAI-compatible models.
---

# 迁移到 GLM 模型

将 Agno 项目中使用 Gemini、OpenAIResponses 或 OpenAIChat 的代码迁移到 GLM 模型（通过 OpenAILike）。

## 适用场景

- 代码使用 `Gemini(id="gemini-3-flash-preview")`，需要切换到 GLM
- 代码使用 `OpenAIResponses(id="gpt-5.2")` 等 OpenAI 系模型，需要切换到 GLM
- 代码使用 `OpenAIChat(id="gpt-4o-mini")` 等 OpenAI Chat 模型，需要切换到 GLM
- 需要统一项目的模型配置为 GLM-4.7 或其他 OpenAI 兼容模型

## 迁移步骤

### 1. 检查当前代码

使用 Grep 查找需要迁移的文件：

```bash
# 来自 Gemini 的文件
Grep pattern="from agno.models.google import Gemini" path="target_dir"
Grep pattern="Gemini(id=" path="target_dir"

# 来自 OpenAIResponses 的文件
Grep pattern="OpenAIResponses(id=" path="target_dir"
Grep pattern="from agno.models.openai import OpenAIResponses" path="target_dir"

# 来自 OpenAIChat 的文件
Grep pattern="OpenAIChat(id=" path="target_dir"
Grep pattern="from agno.models.openai import OpenAIChat" path="target_dir"
```

### 2. 更新单个文件

对每个需要迁移的 Python 文件执行以下操作。

#### 场景 A：从 Gemini 迁移

**A1. 替换 import 语句**

```python
# 旧代码
from agno.models.google import Gemini

# 新代码
import os
from dotenv import load_dotenv
from agno.models.openai import OpenAILike

load_dotenv()
```

**A2. 替换模型配置**

```python
# 旧代码
model=Gemini(id="gemini-3-flash-preview")

# 新代码
model=OpenAILike(
    id=os.getenv("MODEL_ID", "GLM-4.7"),
    base_url=os.getenv("MODEL_BASE_URL", "https://open.bigmodel.cn/api/coding/paas/v4"),
    api_key=os.getenv("MODEL_API_KEY"),
)
```

#### 场景 B：从 OpenAIResponses 迁移

**B1. 替换 import 语句**

```python
# 旧代码
from agno.models.openai import OpenAIResponses

# 新代码
import os
from dotenv import load_dotenv
from agno.models.openai import OpenAILike

load_dotenv()
```

**B2. 替换模型配置**

```python
# 旧代码
model=OpenAIResponses(id="gpt-5.2")

# 新代码
model=OpenAILike(
    id=os.getenv("MODEL_ID", "GLM-4.7"),
    base_url=os.getenv("MODEL_BASE_URL", "https://open.bigmodel.cn/api/coding/paas/v4"),
    api_key=os.getenv("MODEL_API_KEY"),
)
```

#### 场景 C：从 OpenAIChat 迁移

**C1. 替换 import 语句**

```python
# 旧代码
from agno.models.openai import OpenAIChat

# 新代码
import os
from dotenv import load_dotenv
from agno.models.openai import OpenAILike

load_dotenv()
```

**C2. 替换模型配置**

```python
# 旧代码
model=OpenAIChat(id="gpt-4o-mini")

# 新代码
model=OpenAILike(
    id=os.getenv("MODEL_ID", "GLM-4.7"),
    base_url=os.getenv("MODEL_BASE_URL", "https://open.bigmodel.cn/api/coding/paas/v4"),
    api_key=os.getenv("MODEL_API_KEY"),
)
```

**注意：** 同一文件中若有多个 `Agent(...)` 或其它使用模型的地方，每一处 `model=OpenAIChat(...)` 都要替换为上述 OpenAILike 配置。

### 3. 配置环境变量

#### 创建或更新 .env 文件

在项目根目录创建 `.env` 文件：

```env
MODEL_ID=GLM-4.7
MODEL_BASE_URL=https://open.bigmodel.cn/api/coding/paas/v4
MODEL_API_KEY=your-api-key-here
```

#### 确保 .env 在 .gitignore 中

检查 `.gitignore` 包含：

```gitignore
.env
```

### 4. 安装依赖

```bash
uv add python-dotenv openai sqlalchemy yfinance
```

## Embedder 迁移（知识库场景）

如果代码使用了知识库（Knowledge）和向量搜索，需要同时迁移 Embedder。

### 1. 替换 import

```python
# 旧代码
from agno.knowledge.embedder.google import GeminiEmbedder

# 新代码
from agno.knowledge.embedder.openai import OpenAIEmbedder
```

### 2. 替换 Embedder 配置

```python
# 旧代码
embedder=GeminiEmbedder(id="gemini-embedding-001")

# 新代码
embedder=OpenAIEmbedder(
    id=os.getenv("EMBEDDER_MODEL", "embedding-3"),
    api_key=os.getenv("EMBEDDER_API_KEY"),
    base_url=os.getenv("EMBEDDER_BASE_URL", "https://open.bigmodel.cn/api/paas/v4/"),
)
```

### 3. 更新 .env 文件

添加 Embedder 相关环境变量：

```env
# Agent 模型配置
MODEL_ID=GLM-4.7
MODEL_BASE_URL=https://open.bigmodel.cn/api/coding/paas/v4
MODEL_API_KEY=your-api-key

# Embedder 配置（用于知识库向量化）
EMBEDDER_MODEL=embedding-3
EMBEDDER_BASE_URL=https://open.bigmodel.cn/api/paas/v4/
EMBEDDER_API_KEY=your-embedder-api-key
```

**注意：** Embedder 的 `base_url` 使用标准端点（`paas/v4/`），不是 coding 端点。

## 完整示例

### 迁移前

```python
from agno.agent import Agent
from agno.models.google import Gemini
from agno.tools.yfinance import YFinanceTools

agent = Agent(
    name="Finance Agent",
    model=Gemini(id="gemini-3-flash-preview"),
    tools=[YFinanceTools()],
)
```

### 迁移后

```python
import os
from dotenv import load_dotenv

from agno.agent import Agent
from agno.models.openai import OpenAILike
from agno.tools.yfinance import YFinanceTools

load_dotenv()

agent = Agent(
    name="Finance Agent",
    model=OpenAILike(
        id=os.getenv("MODEL_ID", "GLM-4.7"),
        base_url=os.getenv("MODEL_BASE_URL", "https://open.bigmodel.cn/api/coding/paas/v4"),
        api_key=os.getenv("MODEL_API_KEY"),
    ),
    tools=[YFinanceTools()],
)
```

## 批量迁移工作流（推荐）

### 完整流程

#### 1. 查找所有需要迁移的文件

```bash
# 查找 import 语句（按实际使用的模型类选择）
Grep pattern="from agno.models.google import Gemini" path="target_dir"
Grep pattern="from agno.models.openai import OpenAIResponses" path="target_dir"
Grep pattern="from agno.models.openai import OpenAIChat" path="target_dir"

# 统计需要替换的模型实例数量
Grep pattern="model=Gemini" path="target_dir"
Grep pattern="OpenAIResponses\(id=" path="target_dir"
Grep pattern="OpenAIChat\(id=" path="target_dir"
```

#### 2. 逐个文件迁移

对每个文件：
- 先用 Read 读取文件
- 用 Edit 替换 import 语句
- 用 Edit 替换所有模型配置（注意一个文件可能有多处）
- 用 Grep 确认该文件所有模型配置已替换

#### 3. 处理特殊情况

**多 Agent 文件（Team/Workflow）**

文件包含多个 Agent 实例，需要替换所有：

```python
# 示例：11_multi_agent_team.py 有 3 个模型实例
bull_agent = Agent(model=Gemini(...))      # 替换 1
bear_agent = Agent(model=Gemini(...))      # 替换 2
multi_agent_team = Team(model=Gemini(...)) # 替换 3
```

**MemoryManager 模型配置**

```python
# 旧代码
memory_manager = MemoryManager(
    model=Gemini(id="gemini-3-flash-preview"),
    db=agent_db,
)

# 新代码
memory_manager = MemoryManager(
    model=OpenAILike(
        id=os.getenv("MODEL_ID", "GLM-4.7"),
        base_url=os.getenv("MODEL_BASE_URL", "https://open.bigmodel.cn/api/paas/v4/"),
        api_key=os.getenv("MODEL_API_KEY"),
    ),
    db=agent_db,
)
```

#### 4. 验证

```bash
# 验证没有遗漏
Grep pattern="Gemini\(id=" path="target_dir"
Grep pattern="OpenAIResponses\(id=" path="target_dir"
Grep pattern="OpenAIChat\(id=" path="target_dir"
# 以上均应返回：无结果
```

## 注意事项

### 模型兼容性

- **GLM-4.7 不支持 `output_schema`（结构化输出）**
  - 如果代码使用了 `output_schema=SomeModel`，需要移除或改用 `glm-4-flash`
  - 错误表现：返回 Markdown 而非 JSON，导致解析失败

- **工具调用（Tool Calling）**：GLM-4.7 支持，正常工作
- **流式输出（Stream）**：支持
- **多轮对话（History）**：支持

### API 端点

GLM 模型有两个 API 端点：

```python
# 标准端点（推荐）
base_url="https://open.bigmodel.cn/api/paas/v4/"

# Coding 端点（某些场景）
base_url="https://open.bigmodel.cn/api/coding/paas/v4"
```

### 性能对比

| 特性 | Gemini 3 Flash | GLM-4.7 |
|------|----------------|---------|
| 工具调用 | OK | OK |
| 结构化输出 | OK | 不支持 |
| 流式输出 | OK | OK |
| 中文支持 | 好 | 优秀 |
