# agent_as_judge_with_guidelines.py — 实现原理分析

> 源文件：`cookbook/09_evals/agent_as_judge/agent_as_judge_with_guidelines.py`

## 概述

本示例展示 **`AgentAsJudgeEval` 的 `additional_guidelines` 功能**：在基础 `criteria` 之外追加具体检查项，使评判更细粒度和结构化（如要求必须包含单位、应覆盖不同型号等）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `criteria` | "Response should be informative, well-formatted, and accurate for product specifications" | 主要标准 |
| `additional_guidelines` | 3 条具体要求 | 额外检查点 |
| `scoring_strategy` | `"numeric"` | 1-10 分值 |
| `threshold` | `8` | 较高门槛 |

## 核心组件解析

### additional_guidelines 在 prompt 中的注入

`get_evaluator_agent()`（`agent_as_judge.py:198`）在 `additional_guidelines` 非空时追加：

```python
# agent_as_judge.py:245-255（简化）
if self.additional_guidelines:
    instructions_parts.append("## Additional Guidelines")
    for i, guideline in enumerate(self.additional_guidelines, 1):
        instructions_parts.append(f"{i}. {guideline}")
    instructions_parts.append("")
```

最终评判 Agent 的 system prompt 结构：

```
## Criteria
Response should be informative, well-formatted, and accurate for product specifications

## Scoring (1-10)
- 1-2: Completely fails the criteria
...

## Additional Guidelines
1. Must include specific numbers with proper units (mph, km/h, etc.)
2. Should provide context for different model variants if applicable
3. Information should be technically accurate and complete

## Instructions
...
```

### criteria vs additional_guidelines 的设计意图

| 参数 | 用途 | 粒度 |
|------|------|------|
| `criteria` | 整体评判目标，高层次 | 宏观 |
| `additional_guidelines` | 具体检查点，结构化列表 | 微观 |

`criteria` 决定"评什么"，`additional_guidelines` 决定"怎么评"的细节。

## System Prompt 组装（评判 Agent）

```
[system]
You are an expert evaluator. Score outputs objectively based on the provided criteria.

## Criteria
Response should be informative, well-formatted, and accurate for product specifications

## Scoring (1-10)
- 1-2: Completely fails the criteria
- 3-4: Partially meets criteria with significant gaps
- 5-6: Meets basic criteria with notable issues
- 7-8: Meets criteria well with minor issues
- 9-10: Exceeds criteria exceptionally

## Additional Guidelines
1. Must include specific numbers with proper units (mph, km/h, etc.)
2. Should provide context for different model variants if applicable
3. Information should be technically accurate and complete

## Instructions
1. Carefully evaluate the output against the criteria
2. Provide a score from 1 to 10
3. Give a detailed reason for your score
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/agent_as_judge.py` | `additional_guidelines` 字段 L190 | 额外评判指引列表 |
| `agno/eval/agent_as_judge.py` | `get_evaluator_agent()` L245 | guidelines 注入 prompt |
