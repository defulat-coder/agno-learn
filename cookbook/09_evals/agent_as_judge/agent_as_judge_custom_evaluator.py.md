# agent_as_judge_custom_evaluator.py — 实现原理分析

> 源文件：`cookbook/09_evals/agent_as_judge/agent_as_judge_custom_evaluator.py`

## 概述

本示例展示 **`AgentAsJudgeEval` 的自定义 `evaluator_agent` 功能**：用户提供一个完全自定义的 Agent 作为评判者，覆盖内置的 `get_evaluator_agent()` 生成逻辑，适用于需要特定评判风格（如更严格、更宽松）或专业知识领域的场景。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `evaluator_agent` | 自定义 `Agent`（strict evaluator） | 覆盖默认评判 Agent |
| `criteria` | "Explanation must be technically accurate and comprehensive" | 标准（传递给用户定义的 Agent） |
| `scoring_strategy` | `"numeric"` | 需与自定义 Agent 输出格式匹配 |
| `threshold` | `8` | 较高门槛 |

## 核心组件解析

### evaluator_agent 的工作原理

`_evaluate()` 方法（`agent_as_judge.py:273`）中，`evaluator_agent` 参数优先级高于 `get_evaluator_agent()`：

```python
def _evaluate(self, input, output, evaluator_agent=None):
    if evaluator_agent is None:
        evaluator_agent = self.get_evaluator_agent()
    # 直接使用传入的或生成的 evaluator_agent
    response = evaluator_agent.run(prompt)
    ...
```

`run()` 时的调用链：

```python
# agent_as_judge.py:467-490（简化）
evaluator_agent = self.evaluator_agent or self.get_evaluator_agent()
for case in cases:
    self._evaluate(..., evaluator_agent=evaluator_agent)
```

### 自定义 evaluator_agent 的注意事项

自定义 Agent **不需要**设置 `output_schema`（不同于 `AccuracyEval` 的 `evaluator_agent` 必须返回 `AccuracyAgentResponse`）。`AgentAsJudgeEval` 会根据 `scoring_strategy` 动态添加 `output_schema`：

```python
# get_evaluator_agent() 内部
if self.evaluator_agent:
    # 复用用户 Agent，但强制设置 output_schema
    agent = self.evaluator_agent
    agent.output_schema = response_schema  # NumericJudgeResponse 或 BinaryJudgeResponse
    return agent
```

### 自定义 vs 默认评判 Agent 对比

| 特性 | 默认（get_evaluator_agent） | 自定义（evaluator_agent） |
|------|---------------------------|------------------------|
| `description` | "You are an expert evaluator..." | 用户自定义："Strict technical evaluator" |
| `instructions` | 框架生成（含 criteria、scoring） | 用户自定义："Only give high scores to..." |
| `output_schema` | 框架设置 | 框架强制覆盖 |
| 灵活性 | 有限 | 完全自定义 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/agent_as_judge.py` | `evaluator_agent` 字段 L195 | 自定义评判 Agent 注入点 |
| `agno/eval/agent_as_judge.py` | `_evaluate()` L273 | evaluator_agent 选择逻辑 |
| `agno/eval/agent_as_judge.py` | `run()` L479 | `self.evaluator_agent or get_evaluator_agent()` |
