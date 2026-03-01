# agent_as_judge_team_post_hook.py — 实现原理分析

> 源文件：`cookbook/09_evals/agent_as_judge/agent_as_judge_team_post_hook.py`

## 概述

本示例展示 **`AgentAsJudgeEval` 作为 `Team` 的 `post_hooks`** 运行：每次 Team 生成响应后，自动触发评判，无需手动调用 `evaluation.run()`。这是"嵌入式质量监控"模式。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Team Response Quality"` | 评估名称 |
| `model` | `OpenAIChat(id="gpt-5.2")` | 评判 LLM |
| `criteria` | "well-researched, clear, comprehensive, and show good collaboration" | 评判标准 |
| `scoring_strategy` | `"numeric"` | 1-10 分值 |
| `threshold` | `7` | 通过门槛 |
| `Team.post_hooks` | `[agent_as_judge_eval]` | 自动触发 |

## 核心组件解析

### post_hook 触发机制

`AgentAsJudgeEval` 继承自 `BaseEval`，其 `post_check()` 方法是被注册为 post_hook 的关键：

```python
# agent_as_judge.py:841
def post_check(self, run_response: RunResponse, *args, **kwargs):
    """Called automatically by Team after each run."""
    # 从 run_response 提取 input 和 output
    input_text = self._extract_input(run_response)
    output_text = str(run_response.content)
    
    result = self._evaluate(
        input=input_text,
        output=output_text,
        evaluator_agent=self.get_evaluator_agent(),
    )
    self._log_eval_to_db(result)
```

### Team vs Agent post_hook 的差异

| 特性 | Agent post_hook | Team post_hook |
|------|----------------|----------------|
| hook 方法 | `post_check()` | `post_check()` |
| `run_response` 类型 | `RunOutput` | `TeamRunOutput` |
| 输入内容来源 | `run_response.messages[0].content` | Team 的最终问题 |
| DB 记录来源 | `agent_id` | `team.id` |

### DB 结果读取模式

```python
eval_runs = db.get_eval_runs()
latest = eval_runs[-1]
if latest.eval_data and "results" in latest.eval_data:
    result = latest.eval_data["results"][0]
    print(f"Score: {result.get('score', 'N/A')}/10")
```

注意：`eval_data` 是 JSON 序列化后的字典，需通过 `.get()` 方式访问，而不是直接属性访问。

## 完整调用链

```
research_team.run("Explain quantum computing")
  ↓ Team 执行（Researcher + Writer 协作）
  ↓ TeamRunOutput 生成
  ↓ 触发 post_hooks 列表
  └─ agent_as_judge_eval.post_check(run_response)
       ↓ get_evaluator_agent() 构建评判 Agent
       ↓ _evaluate(input, output)
       ↓ NumericJudgeResponse(score=8, reason="...")
       ↓ passed = (8 >= 7) → True
       └─ _log_eval_to_db(db)
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/agent_as_judge.py` | `post_check()` L841 | Team post_hook 入口 |
| `agno/eval/agent_as_judge.py` | `async_post_check()` L887 | 异步 Team post_hook |
| `agno/eval/base.py` | `BaseEval` | post_hook 接口定义 |
