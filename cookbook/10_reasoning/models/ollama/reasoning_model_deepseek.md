# reasoning_model_deepseek.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/ollama/reasoning_model_deepseek.py`

## 概述

本示例展示 **Ollama 上的异模型两阶段推理**：`model=Ollama(llama3.2:latest)`（响应生成）+ `reasoning_model=Ollama(deepseek-r1:14b)`（推理），两个模型均在本地运行，实现本地化的 DeepSeek-R1 两阶段推理。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `Ollama(id="llama3.2:latest")` | 响应生成模型（本地 Llama） |
| `reasoning_model` | `Ollama(id="deepseek-r1:14b", options={"num_predict": 4096})` | 推理模型（本地 DeepSeek-R1） |

## 核心组件解析

### Ollama options 参数

`options={"num_predict": 4096}` 是 Ollama API 特有参数：
- `num_predict` 控制模型生成的最大 token 数（对应其他 API 的 `max_tokens`）
- 对推理模型设置 4096，给予足够的推理空间
- 主模型（llama3.2）未设置 options，使用 Ollama 默认值

### 与 local_reasoning.py 的差异

| 特性 | local_reasoning.py | reasoning_model_deepseek.py |
|------|---------------------|---------------------------|
| 模型数量 | 单模型（单阶段） | 双模型（两阶段） |
| 推理模型 | 无（模型自带推理） | DeepSeek-R1:14B 专属推理 |
| 生成模型 | 相同模型 | Llama 3.2 独立生成 |

### 本地两阶段推理的意义

完全离线的两阶段推理，适合：
- 数据隐私要求严格的场景
- 无互联网访问的本地环境
- 开发测试不消耗 API 额度

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 所有配置 | 均未设置 | — | 否（最简配置） |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/ollama/chat.py` | `Ollama` | Ollama 本地模型接口 |
| `agno/agent/_response.py` | `handle_reasoning()` L70 | 两阶段推理协调 |
