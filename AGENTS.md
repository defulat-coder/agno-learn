# AGENTS.md — Agno

为在此代码库中工作的 AI 编码 Agent 提供的说明。

---

## 仓库结构

```
.
├── libs/agno/agno/          # 核心框架代码
├── cookbook/                # 示例、模式和测试用例（按主题组织）
├── scripts/                 # 开发和构建脚本
├── specs/                   # 设计文档（符号链接，私有）
├── docs/                    # 文档（符号链接，私有）
└── .cursorrules             # 编码模式和约定
```

---

## Conductor 说明

在 Conductor 中工作时，可以使用 `.context/` 目录存放临时笔记或 Agent 间交接产物。该目录已被 gitignore。

---

## 设置符号链接

`specs/` 和 `docs/` 目录从外部位置符号链接。对于新克隆或新工作区，创建以下符号链接：

```bash
ln -s ~/code/specs specs
ln -s ~/code/docs docs
```

这些目录包含未提交到仓库的私有设计文档和说明。

---

## 虚拟环境

本项目使用两个虚拟环境：

| 环境 | 用途 | 安装 |
|-------------|---------|-------|
| `.venv/` | 开发：测试、格式化、验证 | `./scripts/dev_setup.sh` |
| `.venvs/demo/` | Cookbook：包含所有 demo 依赖 | `./scripts/demo_setup.sh` |

**使用 `.venv`** 执行开发任务（`pytest`、`./scripts/format.sh`、`./scripts/validate.sh`）。

**使用 `.venvs/demo`** 运行 cookbook 示例。

---

## 测试 Cookbook

除实现功能外，你最重要的任务是测试和维护 `cookbook/` 目录中的 cookbook。

> 参见 `cookbook/08_learning/` 作为黄金标准。

### 快速参考

**测试环境：**

```bash
# 包含所有依赖的虚拟环境
.venvs/demo/bin/python

# 安装（如需要）
./scripts/demo_setup.sh

# 数据库（如需要）
./cookbook/scripts/run_pgvector.sh
```

**运行 cookbook：**
```bash
.venvs/demo/bin/python cookbook/<folder>/<file>.py
```

### 预期的 Cookbook 结构

每个 cookbook 目录应包含以下文件：
- `README.md` — Cookbook 的说明文档。
- `TEST_LOG.md` — 测试结果日志。

### 测试工作流

**1. 测试前**
- 确保虚拟环境存在（如需要请运行 `./scripts/demo_setup.sh`）
- 启动所需服务（例如 `./cookbook/scripts/run_pgvector.sh`）

**2. 运行测试**
```bash
# 运行单个 cookbook
.venvs/demo/bin/python cookbook/<folder>/<file>.py

# 长时间测试时查看尾部输出
.venvs/demo/bin/python cookbook/<folder>/<file>.py 2>&1 | tail -100
```

**3. 更新 TEST_LOG.md**

每次测试后，使用以下内容更新 cookbook 的 `TEST_LOG.md`：
- 测试名称和路径
- 状态：PASS 或 FAIL
- 测试内容的简要描述
- 任何值得注意的观察或问题

格式：
```markdown
### filename.py

**Status:** PASS/FAIL

**Description:** 测试做了什么以及观察到了什么。

**Result:** 成功/失败摘要。

---
```

---

## 代码位置

| 内容 | 位置 |
|------|-------|
| 核心 Agent 代码 | `libs/agno/agno/agent/` |
| 团队（Teams） | `libs/agno/agno/team/` |
| 工作流（Workflows） | `libs/agno/agno/workflow/` |
| 工具（Tools） | `libs/agno/agno/tools/` |
| 模型（Models） | `libs/agno/agno/models/` |
| 知识库/RAG | `libs/agno/agno/knowledge/` |
| 记忆（Memory） | `libs/agno/agno/memory/` |
| 学习（Learning） | `libs/agno/agno/learn/` |
| 数据库适配器 | `libs/agno/agno/db/` |
| 向量数据库 | `libs/agno/agno/vectordb/` |
| 测试 | `libs/agno/tests/` |

---

## 编码模式

详细模式请参见 `.cursorrules`。核心规则：

- **禁止在循环中创建 Agent** — 复用以提升性能
- **使用 `output_schema`** 获取结构化响应
- **生产环境使用 PostgreSQL**，SQLite 仅用于开发
- **从单个 Agent 开始**，只在必要时扩展
- **同步和异步都要有** — 所有公共方法需要两种变体

---

## 运行代码

**运行 cookbook：**
```bash
.venvs/demo/bin/python cookbook/<folder>/<file>.py
```

**运行测试：**
```bash
source .venv/bin/activate
pytest libs/agno/tests/

# 运行特定测试文件
pytest libs/agno/tests/unit/test_agent.py
```

---

## 实现功能时

1. **在 `specs/` 中查找设计文档** — 如果存在，遵循它
2. **参考现有模式** — 找到类似代码并遵循约定
3. **创建 cookbook** — 每种模式都应有示例
4. **更新 implementation.md** — 标记已完成的内容

---

## 提交代码前

**推送代码或创建 PR 前，务必运行以下脚本：**

```bash
# 首先激活虚拟环境
source .venv/bin/activate

# 格式化所有代码（ruff format）
./scripts/format.sh

# 验证所有代码（ruff check、mypy）
./scripts/validate.sh
```

两个脚本都必须无错误通过才能进行代码审查。

**PR 标题格式：**

PR 标题必须遵循以下格式之一：
- `type: description` — 例如 `feat: add workflow serialization`
- `[type] description` — 例如 `[feat] add workflow serialization`
- `type-kebab-case` — 例如 `feat-workflow-serialization`

有效类型：`feat`、`fix`、`cookbook`、`test`、`refactor`、`chore`、`style`、`revert`、`release`

**PR 描述：**

始终遵循 `.github/pull_request_template.md` 中的 PR 模板。包括：
- 变更摘要
- 变更类型（bug 修复、新功能等）
- 已完成的检查项
- 任何附加背景信息

---

## GitHub 操作

**更新 PR 描述：**

`gh pr edit` 命令可能因与 classic projects 相关的 GraphQL 错误而失败。请改用 API 直接操作：

```bash
# 更新 PR 正文
gh api repos/agno-agi/agno/pulls/<PR_NUMBER> -X PATCH -f body="<PR_BODY>"

# 或使用文件
gh api repos/agno-agi/agno/pulls/<PR_NUMBER> -X PATCH -f body="$(cat /path/to/body.md)"
```

---

## 禁止事项

- 不要在未查看设计文档的情况下实现功能
- 不要在没有变量的 print 行中使用 f-string
- 不要在示例和 print 行中使用表情符号
- 不要跳过公共方法的 async 变体
- 不要在未运行 `./scripts/format.sh` 和 `./scripts/validate.sh` 的情况下推送代码
- 不要在没有详细 PR 描述的情况下提交 PR。始终遵循 `.github/pull_request_template.md` 中提供的 PR 模板。
