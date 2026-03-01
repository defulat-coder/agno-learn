# 向 agno 贡献代码

Agno 是一个开源项目，我们欢迎各种贡献。

## 👩‍💻 如何贡献

请遵循 [fork 和 pull request](https://docs.github.com/en/get-started/quickstart/contributing-to-projects) 工作流：

- Fork 仓库。
- 为你的功能创建一个新分支。
  - 添加你的功能或改进。
  - **确保你的 Pull Request 遵循我们的指南（见下文）。**
  - 提交 pull request。
  - 感谢你的支持与投入！

## Pull Request 指南

为保持清晰有序的项目历史，提交 Pull Request 时请遵守以下指南：

1. **标题格式：** PR 标题必须以方括号括起来的类型标签开头，后跟一个空格和简洁的主题。
   - 示例：`[feat] Add user authentication`
   - 有效类型：`[feat]`、`[fix]`、`[cookbook]`、`[test]`、`[refactor]`、`[chore]`、`[style]`、`[revert]`、`[release]`。
2. **关联 Issue：** PR 描述最好使用 `fixes #<issue_number>`、`closes #<issue_number>` 或 `resolves #<issue_number>` 等关键词引用它所解决的 issue。
   - 示例：`This PR fixes #42 by implementing the new login flow.`

_这些指南由我们的 [PR Lint workflow](.github/workflows/pr-lint.yml) 自动执行。_

## 开发环境配置

1. 克隆仓库。
2. 运行 `uv --version` 检查是否已安装 `uv`。
   - 如果已安装 `uv`，可跳过此步骤。
   - 如果未安装 `uv`，运行 `pip install uv` 进行安装。
3. 创建虚拟环境：
   - Unix 系统使用 `./scripts/dev_setup.sh`。
   - Windows 系统使用 `.\scripts\dev_setup.bat`。
   - 此脚本将：
     - 在当前目录创建 `.venv` 虚拟环境。
     - 安装所需依赖包。
     - 以可编辑模式安装 `agno` 包。
4. 激活虚拟环境：
   - Unix 系统：`source .venv/bin/activate`
   - Windows 系统：`.venv\Scripts\activate`

> 从此步骤起，需使用 `uv pip install` 安装缺失的包

## 格式化与验证

提交 pull request 前，运行相应的格式化和验证脚本，确保代码符合我们的质量标准：

- Unix 系统：
  - `./scripts/format.sh`
  - `./scripts/validate.sh`
- Windows 系统：
  - `.\scripts\format.bat`
  - `.\scripts\validate.bat`

这些脚本将使用 `ruff` 进行代码格式化，并使用 `mypy` 进行静态类型检查。

## 本地测试

提交 pull request 前，确保所有测试在本地通过：

1. 完成上述开发环境配置。

2. 运行测试套件 `./scripts/test.sh`

3. 运行特定测试文件或测试用例：`pytest ./libs/agno/tests/unit/utils/test_string.py` 或你想测试的任意文件。

提交 pull request 前确保所有测试通过。如果添加了新功能，请包含相应的测试覆盖。

## 添加新的向量数据库

1. 按照[开发环境配置](#开发环境配置)设置本地环境。
2. 在 `libs/agno/agno/vectordb` 下为新向量数据库创建新目录。
3. 创建一个实现 `VectorDb` 接口的类：
   - 你的类位于 `libs/agno/agno/vectordb/<your_db>/<your_db>.py` 文件中。
   - `VectorDb` 接口定义在 `libs/agno/agno/vectordb/base.py`
   - 在 `libs/agno/agno/vectordb/<your_db>/__init__.py` 中导入你的 `VectorDb` 类。
   - 参考 [`libs/agno/agno/vectordb/pgvector/pgvector`](https://github.com/agno-agi/agno/blob/main/libs/agno/agno/vectordb/pgvector/pgvector.py) 文件作为示例。
4. 在 `cookbook/07_knowledge/vector_db/<your_db>` 下添加使用你的 `VectorDb` 的 cookbook。
   - 参考 [`cookbook/07_knowledge/vector_db/pgvector/pgvector_db`](https://github.com/agno-agi/agno/blob/main/cookbook/07_knowledge/vector_db/pgvector/pgvector_db.py) 作为示例。
5. 重要：运行 `./scripts/format.sh` 和 `./scripts/validate.sh` 格式化并验证代码。
6. 提交 pull request。

## 添加新的模型提供商

1. 按照[开发环境配置](#开发环境配置)设置本地环境。
2. 在 `libs/agno/agno/models` 下为新模型提供商创建新目录。
3. 如果模型提供商支持 OpenAI API 规范：
   - 创建一个继承 `libs/agno/agno/models/openai/like.py` 中 `OpenAILike` 类的类。
   - 你的类位于 `libs/agno/agno/models/<your_model>/<your_model>.py` 文件中。
   - 在 `libs/agno/agno/models/<your_model>/__init__.py` 中导入你的类。
   - 参考 [`agno/models/together/together.py`](https://github.com/agno-agi/agno/blob/main/libs/agno/agno/models/together/together.py) 文件作为示例。
4. 如果模型提供商不支持 OpenAI API 规范：
   - 在 [Discord](https://discord.gg/4MtYHHrgA8) 上联系我们或提交 issue，讨论集成你的 LLM 提供商的最佳方式。
   - 参考 [`agno/models/anthropic/claude.py`](https://github.com/agno-agi/agno/blob/main/libs/agno/agno/models/anthropic/claude.py) 或 [`agno/models/cohere/chat.py`](https://github.com/agno-agi/agno/blob/main/libs/agno/agno/models/cohere/chat.py) 获取灵感。
5. 将你的模型提供商添加到 `libs/agno/agno/models/utils.py`：
   - 在 `get_model()` 函数中添加新的 `elif` 子句，填入你的提供商名称
   - 使用与你的模块目录匹配的提供商名称（例如 `models/meta/` 对应 "meta"）
   - 导入并返回你的 Model 类，传入提供的 `model_id`
   - 这使用户可以使用字符串格式：`model="yourprovider:model-name"`
   - 示例：
     ```python
     elif provider == "yourprovider":
         from agno.models.yourprovider import YourModel
         return YourModel(id=model_id)
     ```
6. 在 `cookbook/models/<your_model>` 下添加使用你的模型提供商的 cookbook。
   - 参考 [`agno/cookbook/90_models/aws/claude`](https://github.com/agno-agi/agno/tree/main/cookbook/90_models/aws/claude) 作为示例。
   - 在示例中同时展示模型类和字符串语法的用法
7. 重要：运行 `./scripts/format.sh` 和 `./scripts/validate.sh` 格式化并验证代码。
8. 提交 pull request。

## 添加新的工具（Tool）

1. 按照[开发环境配置](#开发环境配置)设置本地环境。
2. 在 `libs/agno/agno/tools` 下为新工具创建新目录。
3. 创建一个继承 `libs/agno/agno/tools/toolkit/toolkit.py` 中 `Toolkit` 类的类：
   - 你的类位于 `libs/agno/agno/tools/<your_tool>.py`。
   - 确保通过 flag 注册类中的所有函数。
   - 参考 [`agno/tools/youtube.py`](https://github.com/agno-agi/agno/blob/main/libs/agno/agno/tools/youtube.py) 文件作为示例。
   - 如果你的工具需要 API key，也可参考 [`agno/tools/serpapi_tools.py`](https://github.com/agno-agi/agno/blob/main/libs/agno/agno/tools/serpapi_tools.py)。
4. 在 `cookbook/tools/<your_tool>` 下添加使用你的工具的 cookbook。
   - 参考 [`agno/cookbook/91_tools/youtube_tools`](https://github.com/agno-agi/agno/blob/main/cookbook/91_tools/youtube_tools.py) 作为示例。
5. 重要：运行 `./scripts/format.sh` 和 `./scripts/validate.sh` 格式化并验证代码。
6. 提交 pull request。

如有任何问题或需要帮助，请在 [Discord](https://discord.gg/4MtYHHrgA8) 上给我们发消息，或在 [Discourse](https://community.agno.com/) 上发帖。

## 📚 资源

- <a href="https://docs.agno.com/introduction" target="_blank" rel="noopener noreferrer">文档</a>
- <a href="https://discord.gg/4MtYHHrgA8" target="_blank" rel="noopener noreferrer">Discord</a>
- <a href="https://community.agno.com/" target="_blank" rel="noopener noreferrer">Discourse</a>

## 📝 许可证

本项目依据 [Apache-2.0 许可证](/LICENSE) 条款授权。
