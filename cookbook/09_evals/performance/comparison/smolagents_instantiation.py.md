# smolagents_instantiation.py — 实现原理分析

> 源文件：`cookbook/09_evals/performance/comparison/smolagents_instantiation.py`

## 概述

本示例展示用 **`PerformanceEval`** 对 **Smolagents `ToolCallingAgent`** 的实例化性能测试，其工具通过继承 `Tool` 基类并实现 `forward()` 方法定义（而非裸函数），迭代 1000 次。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `func` | `instantiate_agent`（Smolagents） | 被测函数 |
| `num_iterations` | `1000` | 高频迭代 |

## 核心组件解析

### 被测函数（Smolagents）

```python
class WeatherTool(Tool):
    name = "weather_tool"
    description = "This is a tool that tells the weather"
    inputs = {"city": {"type": "string", "description": "..."}}
    output_type = "string"

    def forward(self, city: str):
        ...

def instantiate_agent():
    return ToolCallingAgent(
        tools=[WeatherTool()],
        model=InferenceClientModel(model_id="meta-llama/Llama-3.3-70B-Instruct"),
    )
```

Smolagents 使用类继承方式定义工具，每次实例化都需要创建 `WeatherTool()` 对象。

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/eval/performance.py` | `PerformanceEval.run()` L481 | 测量主流程 |
