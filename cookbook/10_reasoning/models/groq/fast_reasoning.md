# fast_reasoning.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/groq/fast_reasoning.py`

## 概述

本示例通过**性能基准测试**展示 **Groq 高速推理**：对比同一模型（Llama 3.3 70B Versatile）在不同模式下的响应时间和推理深度，使用 `time` 模块测量实际响应时长，并通过 `response.reasoning_content` 检查推理深度。

**核心配置对比：**

| 配置项 | 模式1：推理能力模式 | 模式2：标准模式 |
|--------|-----------------|--------------|
| `model` | `Groq(id="llama-3.3-70b-versatile")` | `Groq(id="llama-3.3-70b-versatile")` |
| `markdown` | `True` | `True` |
| `stream` | `False`（计时） | `False`（计时） |

## 核心组件解析

### 推理深度测量

```python
response = agent.run(task, stream=False)
if response.reasoning_content:
    reasoning_len = len(response.reasoning_content.split())
    print(f"Reasoning depth: ~{reasoning_len} words")
```

`reasoning_content` 词数可作为推理深度的量化指标。Groq 在高速推理的同时，通过 LPU 并行计算保持推理深度。

### Groq 速度特点

- Llama 3.3 70B 在 Groq 上的推理速度通常在 200-500 tokens/s
- 两次测试使用相同模型以排除模型差异，单纯对比推理方式的耗时
- 实际场景中 Groq 的优势在延迟（首 token 时间）而非仅吞吐量

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.2.1 | `markdown` | `True` | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/groq/groq.py` | `Groq` | Groq 高速推理模型 |
| `agno/run/response.py` | `RunOutput.reasoning_content` | 推理内容访问 |
