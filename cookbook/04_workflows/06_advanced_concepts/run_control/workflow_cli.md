# workflow_cli.py — 实现原理分析

> 源文件：`cookbook/04_workflows/06_advanced_concepts/run_control/workflow_cli.py`

## 概述

本示例展示 Agno Workflow **`cli_app()` 交互式命令行模式**：`workflow.cli_app()` 启动 REPL 风格的交互式 CLI，用户可在终端连续输入请求，每次输入自动调用 Workflow 并流式输出结果，适合快速原型验证和开发调试。

**核心 API：**

| 参数 | 说明 |
|------|------|
| `input` | 启动时的初始 prompt（可选） |
| `stream=True` | 流式输出响应 |
| `user` | 显示的用户名称 |
| `exit_on` | 触发退出的关键词列表 |

## 核心组件解析

### cli_app 使用

```python
workflow.cli_app(
    input="Create a three-step plan for shipping a workflow feature.",
    stream=True,
    user="Developer",
    exit_on=["exit", "quit"],   # 输入这些词时退出
)
```

### 非交互环境回退

```python
if sys.stdin.isatty():
    # 交互式终端 → CLI 模式
    workflow.cli_app(...)
else:
    # 非交互环境（CI/脚本）→ 单次运行
    workflow.print_response(input=starter_prompt, stream=True)
```

## 关键源码文件索引

| 文件 | 关键类/函数 | 作用 |
|------|------------|------|
| `agno/workflow/workflow.py` | `Workflow.cli_app()` | 交互式 CLI REPL |
