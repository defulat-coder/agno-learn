# local_reasoning.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/ollama/local_reasoning.py`

## 概述

本示例展示**完全本地化的隐私推理**：使用 Ollama 在本地运行两个不同的推理模型（QwQ:32B 和 DeepSeek-R1:8B），所有推理过程不离开本地设备，完全保护数据隐私。

**核心配置对比：**

| 配置项 | QwQ:32B | DeepSeek-R1:8B |
|--------|---------|---------------|
| `model` | `Ollama(id="qwq:32b")` | `Ollama(id="deepseek-r1:8b")` |
| `markdown` | `True` | `True` |
| `show_full_reasoning` | `True` | `True` |

## 核心组件解析

### 本地推理模型对比

| 特性 | QwQ:32B | DeepSeek-R1:8B |
|------|---------|---------------|
| 大小 | 32B 参数 | 8B 参数 |
| 推理质量 | 更高 | 中等 |
| 速度 | 较慢 | 更快 |
| 内存需求 | ~20GB VRAM | ~5GB VRAM |
| 来源 | Alibaba Qwen | DeepSeek |

### 容错处理

```python
try:
    agent.print_response(task, stream=True, show_full_reasoning=True)
except Exception as e:
    print(f"Error: {e}")
    print("Make sure Ollama is running and qwq:32b is installed:")
    print("  ollama pull qwq:32b")
```

两个模型均有完整的错误处理，提示用户检查 Ollama 服务状态和模型安装。

### Ollama 与推理内容

两个本地模型均通过标准格式（`<think>...</think>` 或特殊 token）暴露推理内容，Agno 的 `Ollama` 类将其解析为 `reasoning_content`。

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.2.1 | `markdown` | `True` | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/ollama/chat.py` | `Ollama` | Ollama 本地模型接口 |
