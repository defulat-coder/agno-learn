# debug.py — 实现原理分析

> 源文件：`cookbook/02_agents/14_advanced/debug.py`

## 概述

本示例展示 Agno 的两种 **debug 模式**开启方式：在 Agent 级别设置 `debug_mode=True`（对所有运行生效）或在单次 `run()/print_response()` 调用时传入 `debug_mode=True`（仅对该次生效）。

## 核心配置一览

| 配置项 | 方式 | 作用范围 |
|--------|------|---------|
| `Agent(debug_mode=True)` | Agent 级别 | 所有 run 调用 |
| `agent.print_response(..., debug_mode=True)` | 运行级别 | 单次调用 |

## 核心代码模式

```python
# 方式1：Agent 级别（所有运行都开启 debug）
agent = Agent(
    model=OpenAIResponses(id="gpt-5-mini"),
    debug_mode=True,
)
agent.print_response("Tell me a joke.")

# 方式2：单次运行级别（仅此次开启 debug）
agent = Agent(model=OpenAIResponses(id="gpt-5-mini"))
agent.print_response("Tell me a joke.", debug_mode=True)
```

## debug_mode 效果

开启 `debug_mode` 后，Agno 会输出：
- API 请求体（发送给模型的完整 messages）
- API 响应体（模型的原始返回）
- Tool call 参数和返回值
- System prompt 完整内容
- 计时和 token 信息

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/agent/agent.py` | `Agent(debug_mode=True)` | Agent 级 debug 配置 |
| `agno/agent/agent.py` | `run(debug_mode=True)` | 运行级 debug 覆盖 |
