# agent_as_judge_team.py — 实现原理分析

> 源文件：`cookbook/09_evals/agent_as_judge/agent_as_judge_team.py`

## 概述

本示例展示 **`AgentAsJudgeEval` 对 `Team` 输出的评估**：Team 生成响应后，手动将 `response.content` 传入 `evaluation.run()` 评估。与 `agent_as_judge_team_post_hook.py` 的区别在于：这里是**手动调用**而非 post_hook 自动触发。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| Team 成员 | Researcher + Writer | 两个专责 Agent |
| `criteria` | "well-researched, clear, and comprehensive with good flow" | 评判标准 |
| `scoring_strategy` | `"binary"` | PASSED/FAILED |
| `db` | `SqliteDb`（Team 和 Eval 共享） | 统一 DB |

## 核心组件解析

### 手动评估 Team 输出的流程

```python
# Step 1: Team 执行
response = research_team.run("Explain quantum computing")
# response 是 TeamRunOutput，.content 是最终汇总内容

# Step 2: 手动传入评估
result = evaluation.run(
    input="Explain quantum computing",      # 原始问题
    output=str(response.content),           # Team 的汇总输出
    print_results=True,
    print_summary=True,
)
```

注意：`output=str(response.content)` 仅评估 Team 的**最终输出**，不包含 Researcher 和 Writer 的中间交互内容。若需评估完整协作过程，需使用 `post_hook` 模式（可访问完整 `TeamRunOutput`）。

### 与 agent_as_judge_team_post_hook.py 的对比

| 特性 | 手动调用 | post_hook |
|------|---------|----------|
| 触发方式 | 显式 `evaluation.run(input, output)` | Team 自动触发 |
| 可访问内容 | 仅 `response.content` | 完整 `TeamRunOutput` |
| DB 记录时机 | `evaluation.run()` 调用后 | Team.run() 结束时 |
| 灵活性 | 高（可选择性评估） | 低（每次必评） |

### binary 策略下的 Team 评估

```python
# BinaryJudgeResponse
class BinaryJudgeResponse(BaseModel):
    passed: bool   # Team 输出是否符合标准
    reason: str    # 判断理由（如"content is well-structured and covers...）
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/agent_as_judge.py` | `run()` L467 | 手动调用评估入口 |
| `agno/team/team.py` | `Team.run()` | 返回 `TeamRunOutput` |
| `agno/eval/agent_as_judge.py` | `BinaryJudgeResponse` L34 | Team 输出质量判定 schema |
