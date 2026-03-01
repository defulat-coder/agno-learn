# plan_itinerary.py — 实现原理分析

> 源文件：`cookbook/10_reasoning/models/deepseek/plan_itinerary.py`

## 概述

本示例与 `ethical_dilemma.py` 使用完全相同的架构（`gpt-4o` + `DeepSeek-R1`），但任务场景不同——行程规划（洛杉矶到拉斯维加斯）。展示同一推理模型组合在不同任务类型（旅行规划 vs 伦理分析）上的通用性。

**核心配置一览：**

| 配置项 | 值 | 说明 |
|--------|------|------|
| `model` | `OpenAIChat(id="gpt-4o")` | 响应生成 |
| `reasoning_model` | `DeepSeek(id="deepseek-reasoner")` | 推理模型 |
| `markdown` | `True` | Markdown 格式化 |

## 核心组件解析

### 行程规划中的推理价值

DeepSeek-R1 在行程规划任务中负责：
- 分析距离、路线选项（I-15 等）
- 考虑时间、停靠点、天气等因素
- 生成优先级排序的行程方案

GPT-4o 接收推理结果后，生成格式化的最终行程（Markdown 表格、分段建议）。

## System Prompt 组装

| 序号 | 组成部分 | 值 | 是否生效 |
|------|---------|-----|---------|
| 3.2.1 | `markdown` | `True` | 是 |

## 关键源码文件索引

| 文件 | 关键函数/类 | 作用 |
|------|------------|------|
| `agno/models/deepseek/deepseek.py` | `DeepSeek` | DeepSeek 模型（R1 推理） |
| `agno/models/openai/chat.py` | `OpenAIChat` | GPT-4o 响应生成 |
