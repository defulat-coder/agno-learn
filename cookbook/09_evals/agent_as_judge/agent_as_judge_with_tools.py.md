# agent_as_judge_with_tools.py — 实现原理分析

> 源文件：`cookbook/09_evals/agent_as_judge/agent_as_judge_with_tools.py`

## 概述

本示例展示 **`AgentAsJudgeEval` 评估使用工具的 Agent 输出**：被评估 Agent 使用 `CalculatorTools` 解决数学问题，评判者 Agent 则评估其输出的**可读性和解题过程展示质量**（而非计算正确性本身）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| 被评估 Agent | `gpt-4o` + `CalculatorTools` | 使用工具计算 |
| 评判 Agent | `gpt-5.2` | 评估输出质量 |
| `criteria` | "clearly explain the calculation process, show intermediate steps, and present the final answer in a user-friendly way" | 关注解题过程展示 |
| `scoring_strategy` | `"numeric"` | 1-10 分值 |
| `threshold` | `7` | 通过门槛 |

## 核心组件解析

### 工具 Agent 输出的特殊性

使用工具的 Agent 输出通常包含：
1. 工具调用过程描述
2. 中间计算步骤
3. 最终答案

评判者 Agent 看到的 `output` 是 **Agent 的最终文本响应**（`str(response.content)`），不包含工具调用的原始 JSON。因此，criteria 的重点放在"响应文本中是否清晰展示了计算步骤"。

### 评估的关注点区分

| 评估目标 | 使用框架 |
|---------|---------|
| 工具是否被正确调用 | `ReliabilityEval`（expected_tool_calls） |
| 计算结果是否正确 | `AccuracyEval`（expected_output 对比） |
| 输出质量/可读性 | `AgentAsJudgeEval`（criteria 评判） |

本示例属于第三种，关注输出的**质量和用户体验**，不关注正确性。

### 完整调用链

```
agent.run("What is 15 * 23 + 47?")
  → gpt-4o 决定调用 CalculatorTools.multiply(15, 23) → 345
  → gpt-4o 调用 CalculatorTools.add(345, 47) → 392
  → agent 生成解释性文本回复（含步骤展示）
  → response.content = "Let me calculate this step by step..."

evaluation.run(input="What is 15 * 23 + 47?", output=str(response.content))
  → NumericJudgeResponse(score=8, reason="Response clearly shows calculation steps...")
  → passed = (8 >= 7) → True
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/agent_as_judge.py` | `_evaluate()` L273 | 评判逻辑（输入 output 为文本） |
| `agno/tools/calculator.py` | `CalculatorTools` | 被评估 Agent 使用的工具 |
| `agno/eval/agent_as_judge.py` | `NumericJudgeResponse` L27 | 评判输出 schema |
