# instantiate_team.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/instantiate_team.py`

## 概述

本示例展示 **`PerformanceEval`** 对 **Team 实例化性能**的测量：在 1000 次迭代中测量 `Team(members=[...])` 构造开销，与单 Agent 实例化进行对比参考。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `"Instantiation Performance Team"` | 评估名称 |
| `func` | `instantiate_team` | 被测函数 |
| `num_iterations` | `1000` | 高频迭代 |
| `warmup_runs` | `10`（默认） | 预热 10 次 |

## 核心组件解析

### team_member 在模块级别创建

```python
# 模块级别创建成员 Agent（避免重复创建的开销）
team_member = Agent(model=OpenAIChat(id="gpt-4o"))

def instantiate_team():
    return Team(members=[team_member])  # 复用 team_member
```

> 注意：这里测量的是 `Team.__init__()` 本身的开销，而非成员 Agent 的实例化开销（成员在循环外预先创建）。

### Team 实例化 vs Agent 实例化

| 场景 | 被测内容 |
|------|---------|
| `instantiate_agent.py` | `Agent(system_message=...)` 构造 |
| `instantiate_agent_with_tool.py` | `Agent(model, tools=[...])` 构造 |
| 本文件 | `Team(members=[agent])` 构造 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `PerformanceEval.run()` L481 | 主测量流程 |
| `agno/team/team.py` | `Team.__init__()` | 被测的 Team 构造函数 |
