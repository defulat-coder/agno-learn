# 02_team_streaming_metrics.py — 实现原理分析

> 源文件：`cookbook/03_teams/22_metrics/02_team_streaming_metrics.py`

## 概述

本示例展示 **流式运行的指标获取**：`yield_run_output=True` 在流式事件流末尾追加一个 `TeamRunOutput` 对象，通过 `isinstance(event, TeamRunOutput)` 检测并保存，流结束后从中读取完整指标（包含按模型类型分解的 `metrics.details`）。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `yield_run_output` | `True` | 流末追加 TeamRunOutput |
| `TeamRunOutput` | 从 `agno.run.team` 导入 | 最终运行输出对象 |

## 核心组件解析

### 流式指标获取模式

```python
response = None
for event in team.run("Count from 1 to 5.", stream=True, yield_run_output=True):
    if isinstance(event, TeamRunOutput):
        response = event  # 只在流末收到一次
    # else: 正常处理流式内容...

# 流结束后读取指标
if response and response.metrics:
    pprint(response.metrics)
    
    # 按模型类型分解
    for model_type, model_metrics_list in response.metrics.details.items():
        print(f"{model_type}:")  # "model", "eval_model" 等
        for m in model_metrics_list:
            pprint(m)
```

### `metrics.details` 结构

```python
{
    "model": [ModelMetrics(input_tokens=..., output_tokens=..., time=...)],
    # 如有 eval_model，也会出现在此处
}
```

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/run/team.py` | `TeamRunOutput` | 最终运行输出 |
| `agno/team/team.py` | `run(yield_run_output=True)` | 流末追加输出 |
